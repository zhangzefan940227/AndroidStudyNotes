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

```java
/*
 * Copyright (C) 2021 The Android Open Source Project
 *
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 *      http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */

package com.android.server.pm.permission;

import android.annotation.AppIdInt;
import android.annotation.NonNull;
import android.annotation.Nullable;
import android.annotation.UserIdInt;
import android.content.pm.PackageManager;
import android.content.pm.PermissionGroupInfo;
import android.content.pm.PermissionInfo;
import android.content.pm.permission.SplitPermissionInfoParcelable;
import android.permission.IOnPermissionsChangeListener;
import android.permission.PermissionManager;
import android.permission.PermissionManagerInternal;

import com.android.server.pm.pkg.AndroidPackage;
import com.android.server.pm.pkg.PackageState;

import java.io.FileDescriptor;
import java.io.PrintWriter;
import java.util.List;
import java.util.Map;
import java.util.Set;

/**
 * Interface for managing all permissions and handling permissions related tasks.
 */
public interface PermissionManagerServiceInterface extends PermissionManagerInternal {
    /**
     * Dump.
     */
    void dump(FileDescriptor fd, PrintWriter pw, String[] args);

    /**
     * Retrieve all of the known permission groups in the system.
     *
     * @param flags additional option flags to modify the data returned
     * @return a list of {@link PermissionGroupInfo} containing information about all of the known
     *         permission groups
     */
    List<PermissionGroupInfo> getAllPermissionGroups(
            @PackageManager.PermissionGroupInfoFlags int flags);

    /**
     * Retrieve all of the information we know about a particular group of permissions.
     *
     * @param groupName the fully qualified name (e.g. com.android.permission_group.APPS) of the
     *                  permission you are interested in
     * @param flags additional option flags to modify the data returned
     * @return a {@link PermissionGroupInfo} containing information about the permission, or
     *         {@code null} if not found
     */
    PermissionGroupInfo getPermissionGroupInfo(String groupName,
            @PackageManager.PermissionGroupInfoFlags int flags);

    /**
     * Retrieve all of the information we know about a particular permission.
     *
     * @param permName the fully qualified name (e.g. com.android.permission.LOGIN) of the
     *                       permission you are interested in
     * @param flags additional option flags to modify the data returned
     * @return a {@link PermissionInfo} containing information about the permission, or {@code null}
     *         if not found
     */
    PermissionInfo getPermissionInfo(@NonNull String permName,
            @PackageManager.PermissionInfoFlags int flags, @NonNull String opPackageName);

    /**
     * Query for all of the permissions associated with a particular group.
     *
     * @param groupName the fully qualified name (e.g. com.android.permission.LOGIN) of the
     *                  permission group you are interested in. Use {@code null} to find all of the
     *                  permissions not associated with a group
     * @param flags additional option flags to modify the data returned
     * @return a list of {@link PermissionInfo} containing information about all of the permissions
     *         in the given group, or {@code null} if the group is not found
     */
    List<PermissionInfo> queryPermissionsByGroup(String groupName,
            @PackageManager.PermissionInfoFlags int flags);

    /**
     * Add a new dynamic permission to the system. For this to work, your package must have defined
     * a permission tree through the
     * {@link android.R.styleable#AndroidManifestPermissionTree &lt;permission-tree&gt;} tag in its
     * manifest. A package can only add permissions to trees that were defined by either its own
     * package or another with the same user id; a permission is in a tree if it matches the name of
     * the permission tree + ".": for example, "com.foo.bar" is a member of the permission tree
     * "com.foo".
     * <p>
     * It is good to make your permission tree name descriptive, because you are taking possession
     * of that entire set of permission names. Thus, it must be under a domain you control, with a
     * suffix that will not match any normal permissions that may be declared in any applications
     * that are part of that domain.
     * <p>
     * New permissions must be added before any .apks are installed that use those permissions.
     * Permissions you add through this method are remembered across reboots of the device. If the
     * given permission already exists, the info you supply here will be used to update it.
     *
     * @param info description of the permission to be added
     * @param async whether the persistence of the permission should be asynchronous, allowing it to
     *              return quicker and batch a series of adds, at the expense of no guarantee the
     *              added permission will be retained if the device is rebooted before it is
     *              written.
     * @return {@code true} if a new permission was created, {@code false} if an existing one was
     *         updated
     * @throws SecurityException if you are not allowed to add the given permission name
     *
     * @see #removePermission(String)
     */
    boolean addPermission(PermissionInfo info, boolean async);

    /**
     * Removes a permission that was previously added with
     * {@link #addPermission(PermissionInfo, boolean)}. The same ownership rules apply -- you are
     * only allowed to remove permissions that you are allowed to add.
     *
     * @param permName the name of the permission to remove
     * @throws SecurityException if you are not allowed to remove the given permission name
     *
     * @see #addPermission(PermissionInfo, boolean)
     */
    void removePermission(String permName);

    /**
     * Gets the state flags associated with a permission.
     *
     * @param packageName the package name for which to get the flags
     * @param permName the permission for which to get the flags
     * @param userId the user for which to get permission flags
     * @return the permission flags
     */
    int getPermissionFlags(String packageName, String permName, int userId);

    /**
     * Updates the flags associated with a permission by replacing the flags in the specified mask
     * with the provided flag values.
     *
     * @param packageName The package name for which to update the flags
     * @param permName The permission for which to update the flags
     * @param flagMask The flags which to replace
     * @param flagValues The flags with which to replace
     * @param userId The user for which to update the permission flags
     */
    void updatePermissionFlags(String packageName, String permName, int flagMask,
            int flagValues, boolean checkAdjustPolicyFlagPermission, int userId);

    /**
     * Update the permission flags for all packages and runtime permissions of a user in order
     * to allow device or profile owner to remove POLICY_FIXED.
     */
    void updatePermissionFlagsForAllApps(int flagMask, int flagValues, int userId);

    /**
     * TODO: theianchen We should get rid of the IBinder interface which is an implementation detail
     *
     * Add a listener for permission changes for installed packages.
     * @param listener the listener to add
     */
    void addOnPermissionsChangeListener(IOnPermissionsChangeListener listener);

    /**
     * Remove a listener for permission changes for installed packages.
     * @param listener the listener to remove
     */
    void removeOnPermissionsChangeListener(IOnPermissionsChangeListener listener);

    /**
     * addAllowlistedRestrictedPermission. TODO: theianchen add doc
     */
    boolean addAllowlistedRestrictedPermission(@NonNull String packageName,
            @NonNull String permName, @PackageManager.PermissionWhitelistFlags int flags,
            @UserIdInt int userId);

    /**
     * Gets the restricted permissions that have been allowlisted and the app is allowed to have
     * them granted in their full form.
     * <p>
     * Permissions can be hard restricted which means that the app cannot hold them or soft
     * restricted where the app can hold the permission but in a weaker form. Whether a permission
     * is {@link PermissionInfo#FLAG_HARD_RESTRICTED hard restricted} or
     * {@link PermissionInfo#FLAG_SOFT_RESTRICTED soft restricted} depends on the permission
     * declaration. Allowlisting a hard restricted permission allows for the to hold that permission
     * and allowlisting a soft restricted permission allows the app to hold the permission in its
     * full, unrestricted form.
     * <p>
     * There are four allowlists:
     * <ol>
     * <li>
     * One for cases where the system permission policy allowlists a permission. This list
     * corresponds to the {@link PackageManager#FLAG_PERMISSION_WHITELIST_SYSTEM} flag. Can only be
     * accessed by pre-installed holders of a dedicated permission.
     * <li>
     * One for cases where the system allowlists the permission when upgrading from an OS version in
     * which the permission was not restricted to an OS version in which the permission is
     * restricted. This list corresponds to the
     * {@link PackageManager#FLAG_PERMISSION_WHITELIST_UPGRADE} flag. Can be accessed by
     * pre-installed holders of a dedicated permission or the installer on record.
     * <li>
     * One for cases where the installer of the package allowlists a permission. This list
     * corresponds to the {@link PackageManager#FLAG_PERMISSION_WHITELIST_INSTALLER} flag. Can be
     * accessed by pre-installed holders of a dedicated permission or the installer on record.
     * </ol>
     *
     * @param packageName the app for which to get allowlisted permissions
     * @param flags the flag to determine which allowlist to query. Only one flag can be
     *                      passed.
     * @return the allowlisted permissions that are on any of the allowlists you query for
     * @throws SecurityException if you try to access a allowlist that you have no access to
     *
     * @see #addAllowlistedRestrictedPermission(String, String, int)
     * @see #removeAllowlistedRestrictedPermission(String, String, int)
     * @see PackageManager#FLAG_PERMISSION_WHITELIST_SYSTEM
     * @see PackageManager#FLAG_PERMISSION_WHITELIST_UPGRADE
     * @see PackageManager#FLAG_PERMISSION_WHITELIST_INSTALLER
     */
    List<String> getAllowlistedRestrictedPermissions(@NonNull String packageName,
            @PackageManager.PermissionWhitelistFlags int flags, @UserIdInt int userId);

    /**
     * Removes a allowlisted restricted permission for an app.
     * <p>
     * Permissions can be hard restricted which means that the app cannot hold them or soft
     * restricted where the app can hold the permission but in a weaker form. Whether a permission
     * is {@link PermissionInfo#FLAG_HARD_RESTRICTED hard restricted} or
     * {@link PermissionInfo#FLAG_SOFT_RESTRICTED soft restricted} depends on the permission
     * declaration. Allowlisting a hard restricted permission allows for the to hold that permission
     * and allowlisting a soft restricted permission allows the app to hold the permission in its
     * full, unrestricted form.
     * <p>There are four allowlists:
     * <ol>
     * <li>
     * One for cases where the system permission policy allowlists a permission. This list
     * corresponds to the {@link PackageManager#FLAG_PERMISSION_WHITELIST_SYSTEM} flag. Can only be
     * accessed by pre-installed holders of a dedicated permission.
     * <li>
     * One for cases where the system allowlists the permission when upgrading from an OS version in
     * which the permission was not restricted to an OS version in which the permission is
     * restricted. This list corresponds to the
     * {@link PackageManager#FLAG_PERMISSION_WHITELIST_UPGRADE} flag. Can be accessed by
     * pre-installed holders of a dedicated permission or the installer on record.
     * <li>
     * One for cases where the installer of the package allowlists a permission. This list
     * corresponds to the {@link PackageManager#FLAG_PERMISSION_WHITELIST_INSTALLER} flag. Can be
     * accessed by pre-installed holders of a dedicated permission or the installer on record.
     * </ol>
     * <p>
     * You need to specify the allowlists for which to set the allowlisted permissions which will
     * clear the previous allowlisted permissions and replace them with the provided ones.
     *
     * @param packageName the app for which to get allowlisted permissions
     * @param permName the allowlisted permission to remove
     * @param flags the allowlists from which to remove. Passing multiple flags updates all
     *                       specified allowlists.
     * @return whether the permission was removed from the allowlist
     * @throws SecurityException if you try to modify a allowlist that you have no access to.
     *
     * @see #getAllowlistedRestrictedPermissions(String, int)
     * @see #addAllowlistedRestrictedPermission(String, String, int)
     * @see PackageManager#FLAG_PERMISSION_WHITELIST_SYSTEM
     * @see PackageManager#FLAG_PERMISSION_WHITELIST_UPGRADE
     * @see PackageManager#FLAG_PERMISSION_WHITELIST_INSTALLER
     */
    boolean removeAllowlistedRestrictedPermission(@NonNull String packageName,
            @NonNull String permName, @PackageManager.PermissionWhitelistFlags int flags,
            @UserIdInt int userId);

    /**
     * Grant a runtime permission to an application which the application does not already have. The
     * permission must have been requested by the application. If the application is not allowed to
     * hold the permission, a {@link java.lang.SecurityException} is thrown. If the package or
     * permission is invalid, a {@link java.lang.IllegalArgumentException} is thrown.
     * <p>
     * <strong>Note: </strong>Using this API requires holding
     * {@code android.permission.GRANT_RUNTIME_PERMISSIONS} and if the user ID is not the current
     * user {@code android.permission.INTERACT_ACROSS_USERS_FULL}.
     *
     * @param packageName the package to which to grant the permission
     * @param permName the permission name to grant
     * @param userId the user for which to grant the permission
     *
     * @see #revokeRuntimePermission(String, String, android.os.UserHandle, String)
     */
    void grantRuntimePermission(String packageName, String permName, int userId);

    /**
     * Revoke a runtime permission that was previously granted by
     * {@link #grantRuntimePermission(String, String, android.os.UserHandle)}. The permission must
     * have been requested by and granted to the application. If the application is not allowed to
     * hold the permission, a {@link java.lang.SecurityException} is thrown. If the package or
     * permission is invalid, a {@link java.lang.IllegalArgumentException} is thrown.
     * <p>
     * <strong>Note: </strong>Using this API requires holding
     * {@code android.permission.REVOKE_RUNTIME_PERMISSIONS} and if the user ID is not the current
     * user {@code android.permission.INTERACT_ACROSS_USERS_FULL}.
     *
     * @param packageName the package from which to revoke the permission
     * @param permName the permission name to revoke
     * @param userId the user for which to revoke the permission
     * @param reason the reason for the revoke, or {@code null} for unspecified
     *
     * @see #grantRuntimePermission(String, String, android.os.UserHandle)
     */
    void revokeRuntimePermission(String packageName, String permName, int userId,
            String reason);

    /**
     * Revoke the POST_NOTIFICATIONS permission, without killing the app. This method must ONLY BE
     * USED in CTS or local tests.
     *
     * @param packageName The package to be revoked
     * @param userId The user for which to revoke
     */
    void revokePostNotificationPermissionWithoutKillForTest(String packageName, int userId);

    /**
     * Get whether you should show UI with rationale for requesting a permission. You should do this
     * only if you do not have the permission and the context in which the permission is requested
     * does not clearly communicate to the user what would be the benefit from grating this
     * permission.
     *
     * @param permName a permission your app wants to request
     * @return whether you can show permission rationale UI
     */
    boolean shouldShowRequestPermissionRationale(String packageName, String permName,
            @UserIdInt int userId);

    /**
     * Checks whether a particular permissions has been revoked for a package by policy. Typically
     * the device owner or the profile owner may apply such a policy. The user cannot grant policy
     * revoked permissions, hence the only way for an app to get such a permission is by a policy
     * change.
     *
     * @param packageName the name of the package you are checking against
     * @param permName the name of the permission you are checking for
     *
     * @return whether the permission is restricted by policy
     */
    boolean isPermissionRevokedByPolicy(String packageName, String permName, int userId);

    /**
     * Get set of permissions that have been split into more granular or dependent permissions.
     *
     * <p>E.g. before {@link android.os.Build.VERSION_CODES#Q} an app that was granted
     * {@link Manifest.permission#ACCESS_COARSE_LOCATION} could access the location while it was in
     * foreground and background. On platforms after {@link android.os.Build.VERSION_CODES#Q}
     * the location permission only grants location access while the app is in foreground. This
     * would break apps that target before {@link android.os.Build.VERSION_CODES#Q}. Hence whenever
     * such an old app asks for a location permission (i.e. the
     * {@link PermissionManager.SplitPermissionInfo#getSplitPermission()}), then the
     * {@link Manifest.permission#ACCESS_BACKGROUND_LOCATION} permission (inside
     * {@link PermissionManager.SplitPermissionInfo#getNewPermissions}) is added.
     *
     * <p>Note: Regular apps do not have to worry about this. The platform and permission controller
     * automatically add the new permissions where needed.
     *
     * @return All permissions that are split.
     */
    List<SplitPermissionInfoParcelable> getSplitPermissions();

    /**
     * TODO:theianchen add doc describing this is the old checkPermissionImpl
     */
    int checkPermission(String pkgName, String permName, int userId);

    /**
     * TODO:theianchen add doc describing this is the old checkUidPermissionImpl
     */
    int checkUidPermission(int uid, String permName);

    /**
     * Adds a listener for runtime permission state (permissions or flags) changes.
     *
     * @param listener The listener.
     */
    void addOnRuntimePermissionStateChangedListener(
            @NonNull PermissionManagerServiceInternal
                    .OnRuntimePermissionStateChangedListener listener);

    /**
     * Removes a listener for runtime permission state (permissions or flags) changes.
     *
     * @param listener The listener.
     */
    void removeOnRuntimePermissionStateChangedListener(
            @NonNull PermissionManagerServiceInternal
                    .OnRuntimePermissionStateChangedListener listener);

    /**
     * Get all the package names requesting app op permissions.
     *
     * @return a map of app op permission names to package names requesting them
     */
    Map<String, Set<String>> getAllAppOpPermissionPackages();

    /**
     * Get whether permission review is required for a package.
     *
     * @param packageName the name of the package
     * @param userId the user ID
     * @return whether permission review is required
     */
    boolean isPermissionsReviewRequired(@NonNull String packageName,
            @UserIdInt int userId);

    /**
     * Reset the runtime permission state changes for a package.
     *
     * TODO(zhanghai): Turn this into package change callback?
     *
     * @param pkg the package
     * @param userId the user ID
     */
    void resetRuntimePermissions(@NonNull AndroidPackage pkg,
            @UserIdInt int userId);

    /**
     * Reset the runtime permission state changes for all packages in a user.
     *
     * @param userId the user ID
     */
    void resetRuntimePermissionsForUser(@UserIdInt int userId);

    /**
     * Read legacy permission state from package settings.
     *
     * TODO(zhanghai): This is a temporary method because we should not expose
     * {@code PackageSetting} which is a implementation detail that permission should not know.
     * Instead, it should retrieve the legacy state via a defined API.
     */
    void readLegacyPermissionStateTEMP();

    /**
     * Write legacy permission state to package settings.
     *
     * TODO(zhanghai): This is a temporary method and should be removed once we migrated persistence
     * for permission.
     */
    void writeLegacyPermissionStateTEMP();

    /**
     * Get all the permissions definitions from a package that's installed in the system.
     * <p>
     * A permission definition in a normal app may not be installed if it's overridden by the
     * platform or system app that contains a conflicting definition after system upgrade.
     *
     * @param packageName the name of the package
     * @return the names of the installed permissions
     */
    @NonNull
    Set<String> getInstalledPermissions(@NonNull String packageName);

    /**
     * Get all the permissions granted to a package.
     *
     * @param packageName the name of the package
     * @param userId the user ID
     * @return the names of the granted permissions
     */
    @NonNull
    Set<String> getGrantedPermissions(@NonNull String packageName, @UserIdInt int userId);

    /**
     * Get the GIDs of a permission.
     *
     * @param permissionName the name of the permission
     * @param userId the user ID
     * @return the GIDs of the permission
     */
    @NonNull
    int[] getPermissionGids(@NonNull String permissionName, @UserIdInt int userId);

    /**
     * Get the packages that have requested an app op permission.
     *
     * @param permissionName the name of the app op permission
     * @return the names of the packages that have requested the app op permission
     */
    @NonNull
    String[] getAppOpPermissionPackages(@NonNull String permissionName);

    /** HACK HACK methods to allow for partial migration of data to the PermissionManager class */
    @Nullable
    Permission getPermissionTEMP(@NonNull String permName);

    /** Get all permissions that have a certain protection */
    @NonNull
    List<PermissionInfo> getAllPermissionsWithProtection(
            @PermissionInfo.Protection int protection);

    /** Get all permissions that have certain protection flags */
    @NonNull List<PermissionInfo> getAllPermissionsWithProtectionFlags(
            @PermissionInfo.ProtectionFlags int protectionFlags);

    /**
     * Get all the legacy permissions currently registered in the system.
     *
     * @return the legacy permissions
     */
    @NonNull
    List<LegacyPermission> getLegacyPermissions();

    /**
     * Get the legacy permission state of an app ID, either a package or a shared user.
     *
     * @param appId the app ID
     * @return the legacy permission state
     */
    @NonNull
    LegacyPermissionState getLegacyPermissionState(@AppIdInt int appId);

    /**
     * Read legacy permissions from legacy permission settings.
     *
     * TODO(zhanghai): This is a temporary method because we should not expose
     * {@code LegacyPermissionSettings} which is a implementation detail that permission should not
     * know. Instead, it should retrieve the legacy permissions via a defined API.
     */
    void readLegacyPermissionsTEMP(@NonNull LegacyPermissionSettings legacyPermissionSettings);

    /**
     * Write legacy permissions to legacy permission settings.
     *
     * TODO(zhanghai): This is a temporary method and should be removed once we migrated persistence
     * for permission.
     */
    void writeLegacyPermissionsTEMP(@NonNull LegacyPermissionSettings legacyPermissionSettings);

    /**
     * Callback when the system is ready.
     */
    void onSystemReady();

    /**
     * Callback when a storage volume is mounted, so that all packages on it become available.
     *
     * @param volumeUuid the UUID of the storage volume
     * @param fingerprintChanged whether the current build fingerprint is different from what it was
     *                           when this volume was last mounted
     */
    void onStorageVolumeMounted(@NonNull String volumeUuid, boolean fingerprintChanged);

    /**
     * Get the GIDs computed from the permission state of a UID, either a package or a shared user.
     *
     * @param uid the UID
     * @return the GIDs for the UID
     */
    @NonNull
    int[] getGidsForUid(int uid);

    /**
     * Callback when a user has been created.
     *
     * @param userId the created user ID
     */
    void onUserCreated(@UserIdInt int userId);

    /**
     * Callback when a user has been removed.
     *
     * @param userId the removed user ID
     */
    void onUserRemoved(@UserIdInt int userId);

    /**
     * Callback when a package has been added.
     *
     * @param packageState the added package
     * @param isInstantApp whether the added package is an instant app
     * @param oldPkg the old package, or {@code null} if none
     */
    void onPackageAdded(@NonNull PackageState packageState, boolean isInstantApp,
            @Nullable AndroidPackage oldPkg);

    /**
     * Callback when a package has been installed for a user.
     *
     * @param pkg the installed package
     * @param previousAppId the previous app ID if the package is leaving a shared UID,
     * or Process.INVALID_UID
     * @param params the parameters passed in for package installation
     * @param userId the user ID this package is installed for
     */
    void onPackageInstalled(@NonNull AndroidPackage pkg, int previousAppId,
            @NonNull PermissionManagerServiceInternal.PackageInstalledParams params,
            @UserIdInt int userId);

    /**
     * Callback when a package has been removed.
     *
     * @param pkg the removed package
     */
    void onPackageRemoved(@NonNull AndroidPackage pkg);

    /**
     * Callback when a package has been uninstalled.
     * <p>
     * The package may have been fully removed from the system, or only marked as uninstalled for
     * this user but still installed for other users.
     *
     * @param packageName the name of the uninstalled package
     * @param appId the app ID of the uninstalled package
     * @param packageState the uninstalled package
     * @param pkg the uninstalled package
     * @param sharedUserPkgs the packages that are in the same shared user
     * @param userId the user ID the package is uninstalled for
     */
    void onPackageUninstalled(@NonNull String packageName, int appId,
            @NonNull PackageState packageState, @NonNull AndroidPackage pkg,
            @NonNull List<AndroidPackage> sharedUserPkgs, @UserIdInt int userId);
}
 

```


这里不再贴代码，有兴趣的可以下载aosp源码进行阅读。

