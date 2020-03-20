---
title: RecyclerView建立万能Adapter
tags: Adapter,万能Adapter,RecyclerView
renderNumberedHeading: true
grammar_cjkRuby: true
---


# RecyclerView.Adapter类
RecyclerView.Adapter是一个抽象类：
~~~
/**
     * Base class for an Adapter
     *
     * <p>Adapters provide a binding from an app-specific data set to views that are displayed
     * within a {@link RecyclerView}.</p>
     *
     * @param <VH> A class that extends ViewHolder that will be used by the adapter.
     */
public abstract static class Adapter<VH extends ViewHolder> {}
~~~
该类支持泛型。必须实现三个抽象方法：
public abstract VH onCreateViewHolder(@NonNull ViewGroup parent, int viewType);
public abstract void onBindViewHolder(@NonNull VH holder, int position)
public abstract int getItemCount();
在此之前，需要定义一个ViewHolder继承RecyclerView.ViewHolder。

## onCreateViewHolder
~~~
/**
         * Called when RecyclerView needs a new {@link ViewHolder} of the given type to represent an item.
         * 
         * This new ViewHolder should be constructed with a new View that can represent the items
         * of the given type. You can either create a new View manually or inflate it from an XML
         * layout file.
         * 
         * The new ViewHolder will be used to display items of the adapter using
         * {@link #onBindViewHolder(ViewHolder, int, List)}. Since it will be re-used to display
         * different items in the data set, it is a good idea to cache references to sub views of
         * the View to avoid unnecessary {@link View#findViewById(int)} calls.
         *
         * @param parent The ViewGroup into which the new View will be added after it is bound to an adapter position.
         * @param viewType The view type of the new View.
         *
         * @return A new ViewHolder that holds a View of the given view type.
         * @see #getItemViewType(int)
         * @see #onBindViewHolder(ViewHolder, int)
         */
		 
        @NonNull
        public abstract VH onCreateViewHolder(@NonNull ViewGroup parent, int viewType);
~~~
ViewGroup parent:  可以简单理解为item的根ViewGroup,item的子控件加载在其中。
int viewType： item的类型，可以根据viewType来创建不同的ViewHolder，来加载不同类型的item。

该方法用来创建新的ViewHolder，可以根据需求的itenType，创建多个ViewHolder。创建多个itemType时，需要getItemViewType(int position)方法配合。

## bindViewHolder
~~~
/**
         * Called by RecyclerView to display the data at the specified position. This method should update the contents of the {@link ViewHolder#itemView} to reflect the item at the given position.

         * Note that unlike {@link android.widget.ListView}, RecyclerView will not call this method
         * again if the position of the item changes in the data set unless the item itself is
         * invalidated or the new position cannot be determined. For this reason, you should only
         * use the <code>position</code> parameter while acquiring the related data item inside
         * this method and should not keep a copy of it. If you need the position of an item later
         * on (e.g. in a click listener), use {@link ViewHolder#getAdapterPosition()} which will
         * have the updated adapter position.

         * Override {@link #onBindViewHolder(ViewHolder, int, List)} instead if Adapter can
         * handle efficient partial bind.

         * @param holder The ViewHolder which should be updated to represent the contents of the item at the given position in the data set.
         * @param position The position of the item within the adapter's data set.
         */
		 
        public abstract void onBindViewHolder(@NonNull VH holder, int position);
~~~ 
注意：与ListView的getView不同，只有dataset发生变化时，该方法才会被调用。因此该方法中position不应保存副本。要想获得position，可以通过getAdapterposition方法获得。

## getItemCount
~~~
/**
         * Returns the total number of items in the data set held by the adapter.
         *
         * @return The total number of items in this adapter.
         */
        public abstract int getItemCount();
~~~

# 通用Adapter
## 通用ViewHolder
~~~
package com.vivo.chmusicdemo.base;

import android.content.Context;
import android.util.SparseArray;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.widget.TextView;

import androidx.recyclerview.widget.RecyclerView;

public class ViewHolder extends RecyclerView.ViewHolder {
    private static final String TAG = "commonAdapter";

    private SparseArray<View> mViews;
    private Context mContext;
    private View mConvertView;

    public ViewHolder(Context context, View view) {
        super(view);
        mContext = context;
        mConvertView = view;
        mViews = new SparseArray<>();
    }

    public static ViewHolder onCreateViewHolder(Context context, ViewGroup parent, int layoutId) {
        View view = LayoutInflater.from(context).inflate(layoutId, parent, false);
        return new ViewHolder(context, view);
    }

    public static ViewHolder onCreateViewHolder(Context context, View view) {
        return new ViewHolder(context, view);
    }

    /**
     * 根据ViewID获得控件
     */
    public <T extends View > T getView(int viewId) {
        View view = mViews.get(viewId);
        if(view == null) {
            view = mConvertView.findViewById(viewId);
            mViews.put(viewId, view);
        }
        return (T)view;
    }

    public View getConvertView() {
        return mConvertView;
    }

    /**
     * 以下均为辅助方法
     */
    public void setText(int viewId, String text) {
        TextView textView = getView(viewId);
        textView.setText(text);
    }

    public void setTextColor(int viewId, int textColor) {
        TextView textView = getView(viewId);
        textView.setTextColor(textColor);
    }

}
~~~
辅助方法可以继续添加


## 单一itemViewType
~~~
package com.vivo.chmusicdemo.base;

import android.content.Context;
import android.view.LayoutInflater;

import java.util.List;

public abstract class CommonAdapter<T> extends MultiItemTypeAdapter<T> {
    private static final String TAG = "CommonAdapter";

    private Context mContext;
    private List<T> mDatas;
    private int mLayoutId;
    private LayoutInflater mInflater;

    public CommonAdapter(final Context context, final int layoutId, List<T> datas)
    {
        super(context, datas);
        mContext = context;
        mInflater = LayoutInflater.from(context);
        mLayoutId = layoutId;
        mDatas = datas;

        addDelegate(new ItemViewDelegate<T>()
        {
            @Override
            public int getItemViewLayoutId()
            {
                return layoutId;
            }

            @Override
            public boolean isForViewType( T item, int position)
            {
                return true;
            }

            @Override
            public void convert(ViewHolder holder, T t, int position)
            {
                CommonAdapter.this.convert(holder, t, position);
            }
        });
    }

    protected abstract void convert(ViewHolder holder, T t, int position);
}
~~~



## 多种itemViewtype
多种itemViewType，需要做的是：
1、复写getItemViewType，根据我们的bean 去返回不同的类型
2、onCreateViewHolder中根据itemView生成不同的ViewHolder

新建接口：
~~~
/**
 * 该接口的作用：
 * 0:标识不同的item布局类型
 * 1、返回不同类型的ViewType
 * 2、加载不同类型的Layout，产生不同的ViewHolder
 * @param <T>
 */
public interface ItemViewDelegate<T> {

    /**
     * 获得Layout的Id，用于加载布局
     * @return
     */
    int getItemViewLayoutId();

    /**
     * item是否是需要的类型。
     * 在鸿神的示例代码中，有两种ViewType，因此用boolean标识是否是需要的类型。
     * 个人认为：在ViewType多的情况下，可以用int的返回类型，来标识不同的类型
     * @param item
     * @param position
     * @return
     */
    boolean isForViewType(T item, int position);

    /**
     *配合isForViewType，完成相应的布局工作（setText等）
     * @param viewHolder
     * @param t
     * @param position
     */
    void convert(ViewHolder viewHolder, T t, int position);
}
~~~
管理器：
~~~
package com.vivo.chmusicdemo.base;

import android.util.SparseArray;

public class ItemViewDelegateManager<T> {
    private SparseArray<ItemViewDelegate<T>> mDelegates = new SparseArray<>();

    public int getItemViewCount() {
        return mDelegates.size();
    }

    public ItemViewDelegateManager<T> addDelegate(ItemViewDelegate<T> itemViewDelegate) {
        int viewType = mDelegates.size();
        if(itemViewDelegate != null) {
            mDelegates.put(viewType, itemViewDelegate);
            viewType++;
        }
        return this;
    }

    public ItemViewDelegateManager<T> addDelegate(int viewType, ItemViewDelegate<T> itemViewDelegate) {
        if(mDelegates.get(viewType) != null) {
            throw new IllegalArgumentException(
                    "An ItemViewDelegate is already registered for the viewType = "
                            + viewType
                            + ". Already registered ItemViewDelegate is "
                            + mDelegates.get(viewType));
        }
        mDelegates.put(viewType, itemViewDelegate);
        return this;
    }

    public ItemViewDelegateManager<T> removeDelagate(ItemViewDelegate<T> itemViewDelegate) {
        if(itemViewDelegate == null) {
            throw new NullPointerException("itemViewDelegate is null");
        }
        int indexToRemove = mDelegates.indexOfValue(itemViewDelegate);
        if(indexToRemove > 0) {
            mDelegates.removeAt(indexToRemove);
        }
        return this;
    }

    public ItemViewDelegateManager<T> removeDelegate(int viewType) {
        int indexToRemove = mDelegates.indexOfKey(viewType);
        if(indexToRemove > 0) {
            mDelegates.removeAt(indexToRemove);
        }
        return this;
    }

    public int getItemViewType(T item, int position) {
        int count = mDelegates.size();
        for(int i = count - 1; i >= 0; i--) {
            ItemViewDelegate<T> delegate = mDelegates.valueAt(i);
            if(delegate.isForViewType(item, position)) {
                return mDelegates.keyAt(i);
            }
        }
        throw new IllegalArgumentException(
                "No ItemViewDelegate added that matches position=" + position + " in data source");
    }

    public void convert(ViewHolder viewHolder, T item, int position) {
        int delegateCount = mDelegates.size();
        for(int i = 0; i < delegateCount; i++) {
            ItemViewDelegate<T> delegate = mDelegates.valueAt(i);
            if(delegate.isForViewType(item, position)) {
                delegate.convert(viewHolder, item, position);
                return;
            }
        }
        throw new IllegalArgumentException(
                "No ItemViewDelegateManager added that matches position=" + position + " in data source");
    }

    public ItemViewDelegate<T> getItemViewDelegate(int viewType) {
        return mDelegates.get(viewType);
    }

    public int getItemViewLayoutId(int viewType) {
        return mDelegates.get(viewType).getItemViewLayoutId();
    }

    public int getItemViewType(ItemViewDelegate<T> delegate) {
        return mDelegates.indexOfValue(delegate);
    }
}
~~~
Adapter：
~~~
package com.vivo.chmusicdemo.base;

import android.content.Context;
import android.view.View;
import android.view.ViewGroup;

import java.util.List;

import androidx.recyclerview.widget.RecyclerView;

public class MultiItemTypeAdapter<T> extends RecyclerView.Adapter<ViewHolder> {
    private static final String TAG = "MultiItemTypeAdapter";

    private Context mContext;
    private List<T> mDatas;

    private ItemViewDelegateManager mItemViewDelegateManager;
    private OnItemClickListener mOnItemClickListener;

    MultiItemTypeAdapter(Context context, List<T> datas) {
        mContext = context;
        mDatas = datas;
        mItemViewDelegateManager = new ItemViewDelegateManager<>();
    }

    @Override
    public int getItemViewType(int position) {
        if(!useItemViewDelegateManager()) {
            return super.getItemViewType(position);
        }
        return mItemViewDelegateManager.getItemViewType(mDatas.get(position), position);
    }

    @Override
    public ViewHolder onCreateViewHolder(ViewGroup parent, int viewType) {
        ItemViewDelegate itemViewDelegate = mItemViewDelegateManager.getItemViewDelegate(viewType);
        int layoutId = itemViewDelegate.getItemViewLayoutId();
        ViewHolder viewHolder = ViewHolder.onCreateViewHolder(mContext, parent, layoutId);
        setListener(parent, viewHolder, viewType);
        return viewHolder;
    }

    @Override
    public void onBindViewHolder(ViewHolder viewHolder, int position) {
        convert(viewHolder, mDatas.get(position));
    }

    @Override
    public int getItemCount() {
        return mDatas.size();
    }

    private boolean useItemViewDelegateManager() {
        return mItemViewDelegateManager.getItemViewCount() > 0;
    }

    public void setListener(ViewGroup parent, final ViewHolder viewHolder, final int viewType) {
        if(!isEnable(viewType)) {
            return;
        }
        viewHolder.getConvertView().setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                if(mOnItemClickListener != null) {
                    int position = viewHolder.getAdapterPosition();
                    mOnItemClickListener.onItemClickListener(v, viewHolder, position);
                }
            }
        });

        viewHolder.getConvertView().setOnLongClickListener(new View.OnLongClickListener() {
            @Override
            public boolean onLongClick(View v) {
                if(mOnItemClickListener != null) {
                    int pos = viewHolder.getAdapterPosition();
                    return mOnItemClickListener.onItemLongClickListener(v, viewHolder, pos);
                }
                return false;
            }
        });
    }

    public void convert(ViewHolder viewHolder, T t) {
        mItemViewDelegateManager.convert(viewHolder, t, viewHolder.getAdapterPosition());
    }

    private boolean isEnable(int viewType) {
        return true;
    }

    public List<T> getDatas() {
        return mDatas;
    }

    public MultiItemTypeAdapter addDelegate(ItemViewDelegate<T> delegate) {
        mItemViewDelegateManager.addDelegate(delegate);
        return this;
    }

    public MultiItemTypeAdapter addDelegate(int viewType, ItemViewDelegate<T> delegate) {
        mItemViewDelegateManager.addDelegate(viewType, delegate);
        return this;
    }

    private interface OnItemClickListener{

        void onItemClickListener(View view, RecyclerView.ViewHolder viewHolder, int position);

        boolean onItemLongClickListener(View view, RecyclerView.ViewHolder viewHolder, int position);
    }

    public void setOnItemClickListener(OnItemClickListener listener) {
        mOnItemClickListener = listener;
    }
}

~~~

# HeaderView & FooterView
直接看鸿神的博客：
[Android 优雅的为RecyclerView添加HeaderView和FooterView](https://blog.csdn.net/lmj623565791/article/details/51854533)






# 参考
[Android——RecyclerView入门学习之RecyclerView.Adapter(三)](https://www.jianshu.com/p/b2e6ad1af557)
[为RecyclerView打造通用Adapter 让RecyclerView更加好用](https://blog.csdn.net/lmj623565791/article/details/51118836)
[鸿洋大神的通用Adapter代码](https://github.com/hongyangAndroid/baseAdapter)
[Android 优雅的为RecyclerView添加HeaderView和FooterView](https://blog.csdn.net/lmj623565791/article/details/51854533)