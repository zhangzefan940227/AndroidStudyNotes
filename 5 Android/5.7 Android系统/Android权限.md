# 官方文档
- Android 安全性概览： https://source.android.com/docs/security?hl=zh-cn
- Android 权限： https://source.android.com/docs/core/permissions?hl=zh_cn
- Android 应用架构： https://developer.android.com/topic/architecture/intro?hl=zh-cn


# 权限类型

## 1 安装时权限

安装时权限在用户安装应用时自动授予，如果您在应用中声明了安装时权限，应用商店会在用户查看应用详情页面时向其显示安装时权限通知。

安装时权限子类型，包括**一般权限**和**签名权限。**

**一般权限**

此类权限对用户隐私和其他应用的运行构成的风险很小。

系统会为一般权限分配 `normal` 级别

**签名权限**

只有当应用与定义权限的应用或 OS 使用相同的证书签名时，系统才会向应用授予签名权限。

实现特权服务（如自动填充或 VPN 服务）的应用也会使用签名权限。这些应用需要服务绑定签名权限，以便只有系统可以绑定到该应用的服务。

系统会为签名权限分配 `signature` 保护级别。

## 2 运行时权限

运行时权限也称为危险权限，此类权限授予应用对受限数据的额外访问权限，或允许应用执行对系统和其他应用具有更严重影响的受限操作。

因此，需要先在应用中请求运行时权限，然哼才能访问受限数据或执行受限操作。

当应用请求运行时权限时，系统会显示运行时权限提示，如下图所示。

![image-20250322135133229](./img/image-20250322135133229.png)

 系统会为运行时权限分配 `dangerous` 保护级别。

## 3 特殊权限

特殊权限与特定的应用操作相对应。只有平台和原始设备制造商 (OEM) 可以定义特殊权限。此外，如果平台和 OEM 想要防止有人执行功能特别强大的操作（例如通过其他应用绘图），通常会定义特殊权限。

系统设置中的**特殊应用访问权限**页面包含一组用户可切换的操作。其中的许多操作都是以特殊权限的形式实现的。

系统会为特殊权限分配 `appop` 保护级别。

## 4 权限组

权限可以属于**权限组**。 **权限组由一组逻辑相关的权限组成。**例如，发送和接收短信的权限可能属于同一组，因为它们都涉及应用与短信的互动。

权限组的作用是在应用请求密切相关的多个权限时，帮助系统尽可能减少向用户显示的系统对话框数量。**当系统提示用户授予应用权限时，属于同一组的权限会在同一个界面中显示。** 但是，权限可能会在不另行通知的情况下更改组，因此不要假定特定权限与任何其他权限组合在一起。


# 权限定义

## 1 默认权限
默认权限定义路径：`frameworks/base/core/res/AndroidManifest.xml` 

此外，厂商也可以在ROM中增加额外的自定义权限列表。

## 2 App自定义权限
官方文档： https://developer.android.com/guide/topics/manifest/permission-element?hl=zh-cn

> 声明用于限制对此应用或其他应用的特定组件或功能的访问权限的安全权限。

语法：

```xml
<permission android:description="string resource"
            android:icon="drawable resource"
            android:label="string resource"
            android:name="string"
            android:permissionGroup="string"
            android:protectionLevel=["normal" | "dangerous" |
                                     "signature" | ...] />
```

- `android:description` : 权限的用户可读说明，比标签更长，信息更丰富。例如，当系统询问用户是否向其他应用授予权限时，可能会显示该说明，以向用户说明权限。此属性应设置为对字符串资源的引用。
- `android:icon` : 对表示权限的图标的可绘制资源的引用。
- `android:label` : 权限的用户可读名称。
- `android:name` : 用于在代码中引用权限的名称。例如，在 <uses-permission> 元素或应用组件的 permission 属性中应用

> 注意：
> 系统不允许多个软件包声明同名权限，除非所有软件包均使用同一证书进行签名。
> 如果软件包声明了某个权限，系统不会允许用户安装其他具有相同权限名称的软件包，除非这些软件包使用与第一个软件包相同的证书进行签名。

- `android:permissionGroup` : 将此权限分配给一个组。此属性的值是该组的名称，使用此应用或其他应用中的 <permission-group> 元素声明。如果未设置此属性，则此权限不会属于某个组。
- `android:protectionLevel` : 说明权限中隐含的潜在风险，并指示系统在确定是否将权限授予请求授权的应用时要遵循的流程。

| 值 | 含义 |
| -- | -- |
| `"normal"` | **默认值**。具有较低风险的权限，此类权限允许请求授权的应用访问隔离的应用级功能，对其他应用、系统或用户的风险非常小。系统会自动向在安装时请求授权的应用授予此类权限，无需征得用户的明确许可（但用户始终可以选择在安装之前查看这些权限）。|
| `"dangerous"`	| **具有较高风险的权限，此类权限允许请求授权的应用访问用户私人数据或获取可对用户造成不利影响的设备控制权。** 由于此类权限会带来潜在风险，因此系统可能不会自动向请求授权的应用授予此类权限。例如，应用请求的任何危险权限都可能会向用户显示并且获得确认才会继续执行操作，或者系统会采取一些其他方法来避免用户自动授予使用此类功能的权限。|
| `"signature"`	| **只有在请求授权的应用使用与声明权限的应用相同的证书进行签名时系统才会授予的权限。** 如果证书匹配，系统会在不通知用户或征得用户明确许可的情况下自动授予权限。|
| `"knownSigner"`	| **只有在请求授权的应用使用允许使用的证书进行签名时系统才会授予的权限。** 如果请求者的证书已列出，系统会在不通知用户或征得用户明确许可的情况下自动授予权限。|
| `"signatureOrSystem"`	| `"signature\|privileged"` 的旧同义词。在 API 级别 23 中已废弃。 **系统仅向位于 Android 系统映像的专用文件夹中的应用或使用与声明权限的应用相同的证书进行签名的应用授予的权限。** 不要使用此选项，因为 "signature" 保护级别足以满足大多数需求，无论应用安装在何处，该保护级别都能正常发挥作用。"signatureOrSystem" 权限适用于以下特殊情况：多个供应商将应用内置到一个系统映像中，并且需要明确共享特定功能，因为这些功能是一起构建的。|

# 权限使用

静态授权：在应用需要使用某个权限时，需要在应用对应的 `AndroidManifest.xml` 文件中声明应用所需要的权限。应用安装时授权。

例如：

```xml
<use-permission android:name="android.permission.CAMERA" />
```

动态授权：在应用代码逻辑中，当需要使用某一权限时，往往会先调用 `ContextCompat.checkSelfPermission` 检查应用是否具有某一个权限，如果没有，则会调用 `ActivityCompat.requestPermissions` 请求权限。

例如：

```java
private void requestPermission() {
    if (ContextCompat.checkSelfPermission(this, Manifest.permission.READ_EXTERNAL_STORAGE) != PackageManager.PERMISSION_GRANTED) {
        ActivityCompat.requestPermissions(this, new String[]{Manifest.permission.READ_EXTERNAL_STORAGE}, PERMISSION_REQUEST_CODE);
    }
}
```

# 权限授予

代码路径： `frameworks/base/services/core/java/com/android/server/pm/permission/PermissionManagerServiceImpl.java` 

涉及接口： `restorePermissionState()` `addPermission()` `removePermission()` `updateAllPermissions()` `grantRuntimePermission()` `revokeRuntimePermission()` 等

PermissionManagerServiceImpl 实现了 `frameworks/base/services/core/java/com/android/server/pm/permission/PermissionManagerServiceInterface.java` 接口，具体对外开放的接口可以查看 `PermissionManagerServiceInterface` 中的注释。

这里不再贴代码，有兴趣的可以下载aosp源码进行阅读。

