#  主要作用

| 组件           | 作用                                                         |
| -------------- | ------------------------------------------------------------ |
| Room（数据库） | 提供 SQLite 的抽象层，管理本地数据存储                       |
| ViewModel      | 管理 UI 相关的数据，确保数据在配置变更（如旋转屏幕）时不会丢失 |
| LiveData       | 观察数据变化，并在 UI 组件生命周期内自动更新                 |



#  完整示例

## **功能**

我们实现一个**简单的待办事项（ToDo List）应用**，可以：

- **增删查改任务**，数据存储在 `Room` 数据库中。
- **使用 `LiveData`** 让 UI **自动更新**，而不需要手动刷新。
- **`ViewModel` 负责管理数据**，避免 `Activity` 直接操作数据库。



## 创建 Room 数据库

### （1）定义 Entity（数据库表）

```kotlin
import androidx.room.Entity
import androidx.room.PrimaryKey

@Entity(tableName = "todo_table")
data class Todo(
    @PrimaryKey(autoGenerate = true) val id: Int = 0,
    val task: String,
    val isCompleted: Boolean
)
```

- `@Entity`：表示这是一个数据库表，表名是 `todo_table`。
- `@PrimaryKey(autoGenerate = true)`：`id` 是主键，`Room` 会自动生成。



### （2）创建DAO(数据访问对象）

```kotlin
import androidx.lifecycle.LiveData
import androidx.room.*

@Dao
interface TodoDao {
    @Insert(onConflict = OnConflictStrategy.REPLACE)
    suspend fun insert(todo: Todo)

    @Update
    suspend fun update(todo: Todo)

    @Delete
    suspend fun delete(todo: Todo)

    @Query("SELECT * FROM todo_table ORDER BY id DESC")
    fun getAllTodos(): LiveData<List<Todo>> // 使用 LiveData 让 UI 自动更新
}
```

- `@Dao`：数据访问对象，定义数据库操作。
- `@Insert` / `@Update` / `@Delete`：操作数据库的基本增、删、改。
- `getAllTodos()` 返回 `LiveData<List<Todo>>`，**Room 会自动监听数据变化，并更新 UI**。



###  （3）创建 Room 数据库

```kotlin
import android.content.Context
import androidx.room.Database
import androidx.room.Room
import androidx.room.RoomDatabase

@Database(entities = [Todo::class], version = 1, exportSchema = false)
abstract class TodoDatabase : RoomDatabase() {
    abstract fun todoDao(): TodoDao

    companion object {
        @Volatile
        private var INSTANCE: TodoDatabase? = null

        fun getDatabase(context: Context): TodoDatabase {
            return INSTANCE ?: synchronized(this) {
                val instance = Room.databaseBuilder(
                    context.applicationContext,
                    TodoDatabase::class.java,
                    "todo_database"
                ).build()
                INSTANCE = instance
                instance
            }
        }
    }
}
```

- `@Database(entities = [Todo::class], version = 1)`：定义数据库，包含 `Todo` 表。
- `synchronized(this)`：保证数据库实例的线程安全。
- `Room.databaseBuilder()`：构建 `Room` 数据库实例。



##  **创建Repository**

`Repository` 负责处理 `DAO` 操作，避免 `ViewModel` 直接操作数据库。

```kotlin
import androidx.lifecycle.LiveData

class TodoRepository(private val todoDao: TodoDao) {
    val allTodos: LiveData<List<Todo>> = todoDao.getAllTodos()

    suspend fun insert(todo: Todo) {
        todoDao.insert(todo)
    }

    suspend fun update(todo: Todo) {
        todoDao.update(todo)
    }

    suspend fun delete(todo: Todo) {
        todoDao.delete(todo)
    }
}
```

- `Repository` 封装 `DAO`，**ViewModel 只需要调用 `Repository`，而不直接操作 `Room`**。



## 创建 ViewModel

```kotlin
import androidx.lifecycle.*
import kotlinx.coroutines.launch

class TodoViewModel(private val repository: TodoRepository) : ViewModel() {
    val allTodos: LiveData<List<Todo>> = repository.allTodos

    fun insert(todo: Todo) = viewModelScope.launch {
        repository.insert(todo)
    }

    fun update(todo: Todo) = viewModelScope.launch {
        repository.update(todo)
    }

    fun delete(todo: Todo) = viewModelScope.launch {
        repository.delete(todo)
    }
}
```

- `viewModelScope.launch {}` 让数据库操作在 **后台线程执行**（`suspend` 需要 `Coroutine`）。
- `LiveData<List<Todo>>` 确保 UI 自动更新。



## 创建 ViewModelFactory

```kotlin
import androidx.lifecycle.ViewModel
import androidx.lifecycle.ViewModelProvider

class TodoViewModelFactory(private val repository: TodoRepository) : ViewModelProvider.Factory {
    override fun <T : ViewModel> create(modelClass: Class<T>): T {
        if (modelClass.isAssignableFrom(TodoViewModel::class.java)) {
            @Suppress("UNCHECKED_CAST")
            return TodoViewModel(repository) as T
        }
        throw IllegalArgumentException("Unknown ViewModel class")
    }
}
```

- `ViewModelFactory` 用于 **传递 `Repository`**，让 `ViewModel` 使用 `Room`。



## 在Activity里使用

```kotlin
class MainActivity : AppCompatActivity() {
    private lateinit var todoViewModel: TodoViewModel

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        // 获取数据库实例和 DAO
        val database = TodoDatabase.getDatabase(this)
        val repository = TodoRepository(database.todoDao())

        // 创建 ViewModel
        val factory = TodoViewModelFactory(repository)
        todoViewModel = ViewModelProvider(this, factory).get(TodoViewModel::class.java)

        // 监听 LiveData，更新 UI
        todoViewModel.allTodos.observe(this) { todos ->
            // 更新 RecyclerView 或 UI
        }

        // 添加任务
        buttonAdd.setOnClickListener {
            val newTodo = Todo(task = "New Task", isCompleted = false)
            todoViewModel.insert(newTodo)
        }
    }
}
```

- `getDatabase(this)` 获取数据库实例。
- `ViewModelProvider(this, factory)` 创建 `ViewModel`。
- `LiveData.observe()` 监听数据变化，**UI 自动更新**。



## 总结

✅ **Room 负责存储数据**，`DAO` 负责数据库操作。 

✅ **ViewModel 负责持久化数据**，避免因屏幕旋转导致数据丢失。 

✅ **LiveData 让 UI 自动更新**，不需要手动刷新数据。 

✅ **Repository 作为数据中介**，避免 `ViewModel` 直接操作 `Room`。

**最终效果**：

1. **数据库变更时，UI 自动更新**（`LiveData`）。
2. **ViewModel 让数据在 Activity/Fragment 生命周期内持久化**。
3. **Repository 让数据管理更清晰**，避免 `ViewModel` 直接访问数据库。