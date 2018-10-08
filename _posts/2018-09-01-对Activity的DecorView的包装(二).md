---
title: 对Activity的DecorView的包装(二)
author: luo0612
layout: post
---

看了下公司的系统代码对于根布局decor_layout.xml的修改, 有所获.  

前些时候才开始做系统开发的时候, 总想改改系统的源码, 至于原因: 人总是想装装, 在踩过几个别人修改的坑后, 还是觉得在不改源码的基础上, 尽量纳源码为自己所用还是挺好的.

**代码如下:**

    public void wrapDecor(Activity activity) {

        mWindow = activity.getWindow();
        if(mWindow == null){
            Log.e(TAG, "Window is null");
            return;
        }
        
        View decorView = mWindow.getDecorView();
        if (decorView == null) {
            Log.e(TAG, "DecorView is null");
            return;
        }

        ViewGroup contentView = (ViewGroup) decorView.findViewById(android.R.id.content);
        if (contentView == null) {
            Log.e(TAG, "DecorView is null, have you called wrapDecor after Activity#super.onCreate?");
            return;
        }

        final int childCount = contentView.getChildCount();
        if (childCount == 0) {
            // Maybe called before Activity#setContentView
            mPotentialErrorFlag |= FLAG_POTENTIAL_ERROR_SET_CONTENT;
        }

        View[] children = new View[childCount];
        for (int i = 0; i < childCount; i++) {
            children[i] = contentView.getChildAt(i);
        }

        contentView.removeAllViews();

        LayoutInflater inflater = LayoutInflater.from(activity);

		//===================== begin ========================

		// 此处即为自定义的decor_layout.xml文件
        View wrapper = inflater.inflate(R.layout.decor_layout, null);

        ViewGroup rawContentView = (ViewGroup) wrapper.findViewById(R.id.content);
        if (childCount > 0) {
            for (View child : children) {
                rawContentView.addView(child);
            }
        }
        //change for listActivity, add view first then setContenView
        activity.setContentView(wrapper);

		//=====================   end   =======================

		// 获取自定义decor_layout中的控件
        mOptionsKey = wrapper.findViewById(R.id.feature_bar_options);

		// 此处获取的是ActionBar的控件, 由于项目中需要大量使用到ActionBar, 
		// 此处对覆盖ActionBar对OptionMenu的控制
        ActionBarView actionBarView = (ActionBarView) decorView.findViewById(
                com.android.internal.R.id.action_bar);
        if (actionBarView != null) {
			// 覆盖ActionBar对OptionMenu的控制
            actionBarView.setOverrideOverflowButton(mOptionsKey);
        } else {
            Log.d(TAG, "actionBarView is null");
            if (mWindow != null) {
                Log.d(TAG, "Attempt to invoke setShouldOverrideResources access PhoneWindow");
                mWindow.setShouldOverrideResources(true);
            } else {
                 Log.d(TAG, "mWindow is empty, pls check it");
            }
        }
    }


该段代码的核心, 就在上面的 begin 和 end 之间, 代码挺简单, 使用到包装的思想, 也就是包装设计模式.




