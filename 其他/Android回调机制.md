# Android 回调机制



回调本质上还是方法调用。只是为了防止两个类互相持有对方的对象导致耦合度过高。



#### 看一个例子



- class A 实现接口 CallBack callback ——背景1

- class A 中包含一个 class B 的引用 b ——背景2

- class B 有一个形参为 callback 的方法 f(CallBack callback) ——背景3

  A 的对象 a 调用 B 的方法 f(CallBack callback) ——A 类调用 B 类的某个方法 C

  然后 b 就可以在 f(CallBack callback) 方法中调用A的方法 ——B 类调用 A 类的某个方法 D



```java
/*定义一个接口Callback*/
interface Callback{
    void doCallback(String result);
}
/*A类实现接口*/
class A implements Callback{
/*A类持有B类的引用，并把b传入A*/
    B b;
    A(B b){
        this.b = b;
    }
    /*定义A类方法*/
    public void funtionA(){
        //此时b方法传入的参数必须是Callback的子类，这里我们传入A，因为A实现了Callback
        b.setOnClickListener(A.this);
    }
    
    @Override
    public void doCallback(String result) {
        System.out.println("the answer is:"+result);
        
    }

}

class B {
    //创建一个含有Callback实例的B类方法
    public void setOnClickListener(Callback callback){
        String answer = "callback";
        /*B类方法中调用A类方法，这个A类方法要包括接口中的那个抽象方法，即doCallback(),并将所得结果作为参数传回A类中
         * 这里就是回调，实现内容在B类的回调中完成
         */
         callback.doCallback(answer);
    }
}

public class CallBackTest{
    public static void main(String [] args){
        B b = new B();
        A a = new A(b);
        //调用A类方法进回调
        a.funtionA();
    }
}
```



流程分析：



- 主函数执行到a.funtionA();
- 调用b.setOnClickListener(a);
- String answer = "callback";表示在b中完成了该流程得到结果
- a.doCallback(answer);然后将结果以参数的形式传入a中，执行doCallback()方法，回调
- System.out.println("the answer is:"+result);[//a.doCallback](http://a.doCallback)(answer);执行后进行结果输出



在实际应用中，最常用的 onClick() 就是一种回调机制。看实例



#### 第一种 onClick



这个是View类的回调接口



```java
//这个是View的一个回调接口  
/** 
 * Interface definition for a callback to be invoked when a view is clicked. 
 */  
public interface OnClickListener {  
    /** 
     * Called when a view has been clicked. 
     * 
     * @param v The view that was clicked. 
     */  
    void onClick(View v);  
}
```



**MainActivity 相当于 A 类**



```java
import android.app.Activity;  
import android.os.Bundle;  
import android.view.View;  
import android.view.View.OnClickListener;  
import android.widget.Button;  
import android.widget.Toast;  

public class MainActivity extends Activity implements OnClickListener{  
    /** 
     * Class A 包含Class B的引用----->背景二 
     */  
    private Button button;  
  
    @Override  
    public void onCreate(Bundle savedInstanceState) {  
        super.onCreate(savedInstanceState);  
        setContentView(R.layout.activity_main);  
        button = (Button)findViewById(R.id.button1);  
          
        /** 
         * Class A 调用View的方法,而Button extends View----->A类调用B类的某个方法 C 
         */  
        button.setOnClickListener(this);  
    }  
  
    /** 
     * 用户点击Button时调用的回调函数，你可以做你要做的事 
     * 这里我做的是用Toast提示OnClick 
     */  
    @Override  
    public void onClick(View v) {  
        Toast.makeText(getApplication(), "OnClick", Toast.LENGTH_LONG).show();  
    }  
  
}
```



**View 类（原生类）相当于B类**



```java
public class View implements Drawable.Callback, KeyEvent.Callback, AccessibilityEventSource {  
    /** 
     * Listener used to dispatch click events. 
     * This field should be made private, so it is hidden from the SDK. 
     * {@hide} 
     */  
    protected OnClickListener mOnClickListener;  
      
    /** 
     * setOnClickListener()的参数是OnClickListener接口------>背景三 
     * Register a callback to be invoked when this view is clicked. If this view is not 
     * clickable, it becomes clickable. 
     */  
      
    public void setOnClickListener(OnClickListener l) {  
        if (!isClickable()) {  
            setClickable(true);  
        }  
        mOnClickListener = l;  
    }  
      
      
    /** 
     * Click的触发是在系统捕捉到ACTION_UP后发生并由performClick()执行的，
     *performClick里会调用先前注册的监听器的onClick()方法：      
     */  
    public boolean performClick() {  
        sendAccessibilityEvent(AccessibilityEvent.TYPE_VIEW_CLICKED);  
  
        if (mOnClickListener != null) {  
            playSoundEffect(SoundEffectConstants.CLICK);  
              
            //这个相当于B类调用A类的某个方法D，这个D就是所谓的回调方法咯  
            mOnClickListener.onClick(this);  
            return true;  
        }  
  
        return false;  
    }  
}
```



#### 第二种 onClick()



**Button 为 B 类（子类），重写了 A类 （父类）View类的 setOnClickListener() 方法**



```java
button.setOnClickListener(new View.OnClickListener() {
    @Override 
    //注意：此时传入的参数是父类v，也就是说，会将执行结果返回给view
    public void onClick(View v) { 
    //做一些操作 doWork(); 
    }
});
```



View中的代码



```java
//这里将OnClickListener赋值给了mOnClickListene
public void setOnClickListener (@Nullable OnClickListener l) { 
    if (!isClickable()) { 
        setClickable(true); 
    } 
    getListenerInfo().mOnClickListener = l; 

}

public boolean performClick() { 
    sendAccessibilityEvent(AccessibilityEvent.TYPE_VIEW_CLICKED); 
    ListenerInfo li = mListenerInfo; 
    if (li != null && li.mOnClickListener != null) { 
        playSoundEffect(SoundEffectConstants.CLICK); 
        li.mOnClickListener.onClick(this);
        return true; 
    } 
    return false; 
}
```



触发点击事件后系统自动执行 View 中的 perfromClick() 方法，自然也就执行到了 mOnClickListener.onClick(view)



还是由 View 中 onClick() 调用



两种写法的差别在于：

第一种写法：A类为MainActivity.class,B类为View（Button继承View）

第二种写法：A类为View类，B类为Button类