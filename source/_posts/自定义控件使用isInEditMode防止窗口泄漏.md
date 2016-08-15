title: 自定义控件使用isInEditMode防止窗口泄漏
date: 2015-08-12 15:35:14
tags: [Android]
---
在开发过程中有时候会遇到Activity has leaked window.，此时便是出现了窗口泄漏问题。

Android的每一个Activity都有个WindowManager窗体管理器，同样构建在某个Activity之上的对话框、PopupWindow也有相应的WindowManager窗体管理器。因为对话框、PopupWindow不能脱离Activity而单独存在着，所以当某个Dialog或者某个PopupWindow正在显示的时候我们去finish()了承载该Dialog或PopupWindow的 Activity时，就会抛WindowLeaked异常了，因为这个Dialog或PopupWindow的WindowManager已经没有谁可以附属了，所以它的窗体管理器已经泄漏了。

**使用isInEditMode解决可视化编辑器无法识别自定义控件的问题**


> isInEditMode:
Indicates whether this View is currentlyin edit mode. A View is usually in edit mode when displayed within a developertool. For instance, if this View is being drawn by a visual user interfacebuilder, this method should return true. Subclasses should check the returnvalue of this method to provide different behaviors if their normal behaviormight interfere with the host environment. For instance: the class spawns athread in its constructor, the drawing code relies on device-specific features,etc. This method is usually checked in the drawing code of custom widgets.

所以可以在自定义view的初始化或者需要加载界面的地方设置此方法，避免导致窗体泄漏的问题。

  	private void init() {
    	if (isInEditMode()) {
      	return;
    	}
		//初始化代码
  	}