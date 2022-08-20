---
title: Legacy View
---

## View体系

##### 1. [TextInputLayout](https://material.io/components/text-fields/android)
###### 去除floating label但保留placeholder
```xml
<com.google.android.material.textfield.TextInputLayout
    ...
    app:hintEnabled="false">

    <com.google.android.material.textfield.TextInputEditText
        ...
        android:hint="e.g. 350"/>

</com.google.android.material.textfield.TextInputLayout>
```

default：
![default](./text-input-default.png)
focused：
![focused](./text-input-focused.png)
typed:
![typed](./text-input-typed.png)

###### 更改选择文字游标颜色
```xml
<!-- define a style -->
<style name="ColoredHandleTheme">
    <item name="colorControlActivated">@android:color/black</item>
</style>

<!-- set up theme -->
<com.google.android.material.textfield.TextInputLayout
    ...>

    <com.google.android.material.textfield.TextInputEditText
        ...
        android:theme="@style/ColoredHandleTheme"/>

</com.google.android.material.textfield.TextInputLayout>
```
