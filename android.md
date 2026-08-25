# 核心理念与原则
> **简洁至上**：恪守KISS（Keep It Simple, Stupid）原则，崇尚简洁与可维护性，避免过度工程化与不必要的防御性设计。
> **深度分析**：立足于第一性原理（First Principles Thinking）剖析问题，并善用工具以提升效率。
> **事实为本**：以事实为最高准则。若有任何谬误，恳请坦率斧正，助我精进。

# 开发工作流
> **渐进式开发**：通过多轮对话迭代，明确并实现需求。在着手任何设计或编码工作前，必须完成前期调研并厘清所有疑点。
> **结构化流程**：严格遵循“构思方案 → 提请审核 → 分解为具体任务”的作业顺序。

## 验证规则
> 完成代码修改后默认只进行静态检查，不执行 Gradle 构建。
> 不安装 APK，不使用模拟器或真机进行验证。
> 不使用 Android CLI、ADB 等命令行工具验证。
> 只有用户明确要求时，才执行上述验证操作。

# 输出规范
> **语言要求**：所有回复、思考过程及任务清单，均须使用中文，implementation_plan.md等md文件均须使用中文
> **固定指令**：\Implementation Plan, Task List and Thought in Chinese\ 相关指令已整合至本规范中。

---

# Android Java 项目开发规范

# 项目概述

这是一个 Android Java 项目，使用 DataBinding/ViewBinding、Room 数据库、RecyclerView 等组件。

# Activity 开发规范

## 基类使用

- **所有 Activity 必须继承**`BaseActivity<Binding, Presenter>`

- 泛型参数：第一个是 DataBinding 类（如 `ActivityContactListBinding`），第二个是 Presenter（通常为 `BasePresenter`）

## 生命周期方法顺序

```java

@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setDataBindingLayout(R.layout.activity_xxx);  // 必须在 onCreate 中设置布局
}

@Override
public void initView() {
    // 初始化视图、标题栏、工具栏、RecyclerView 等
    binding.include.setTitleStr("标题");
}

@Override
public void bindEvent() {
    // 绑定事件监听器
    binding.setOnClickListener(this::onClickCallback);
}

@Override
public void onClickCallback(View view) {
    int id = view.getId();
    if (id == R.id.iv_title_back) {
        finish();
    } else if (id == R.id.tv_title_right) {
        saveData();
    }
}
```

## 工具栏初始化

- 使用 `binding.include.setTitleStr(String title)` 设置标题


## 视图访问

- **必须使用 `binding` 变量访问视图**，如 `binding.rv`、`binding.tvEmpty`

- 不要使用 `findViewById()`

## RecyclerView 设置

```java

adapter = new XxxAdapter(this, list, R.layout.item_xxx);
binding.rv.setLayoutManager(new LinearLayoutManager(this));
binding.rv.addItemDecoration(new SimplePaddingDecoration(this, SizeUtils.dp2px(0)));  // 设置间距
binding.rv.setAdapter(adapter);
// 或者使用
adapter = new XxxAdapter(this, list, R.layout.item_xxx);
binding.rv.setLayoutManager(new GridLayoutManager(this, 2));
binding.rv.addItemDecoration(new GridSpacesItemDecoration(2, SizeUtils.dp2px(8f), false));
binding.rv.setAdapter(adapter);
```

## 数据加载

- 在 `initView()` 中调用 `getData()` 方法刷新数据

# Fragment 开发规范

## 基类使用

- **所有 Fragment 必须继承**`BaseFragment<Binding, Presenter>`

- 提供静态 `newInstance()` 方法创建实例


## 必须实现的方法

```java

public static XxxFragment newInstance() {
    return new XxxFragment();
}

@Override
public int onSetLayoutId() {
    return R.layout.fra_xxx;
}

@Override
public void initView() {
    // 初始化视图
}

@Override
public void bindEvent() {
    binding.setOnClickListener(this::onClickCallback);
}

@Override
public void onClickCallback(View view) {
    int id = view.getId();
    if (id == R.id.iv_title_back) {
        finish();
    } else if (id == R.id.tv_title_right) {
        saveData();
    }
}
```

# Adapter 开发规范

## 基类使用

- **所有 Adapter 必须继承**`BaseRecylerAdapter<Entity>`

- Entity 是数据实体类

## 标准结构

```java

public class XxxListAdapter extends BaseRecylerAdapter<XxxEntity> {
    private Context context;
    private OnMoreClickListener onMoreClickListener;

    // 构造函数
    public XxxListAdapter(Context context, List<XxxEntity> list, int layoutId) {
        super(context, list, layoutId);
        this.context = context;
    }

    // 绑定数据
    @Override
    public void convert(MyRecylerViewHolder holder, int position) {
        XxxEntity entity = mDatas.get(position);
        
        // 使用 holder 设置视图
        holder.setText(R.id.tv_name, entity.getName());
    }
}
```

## Adapter 使用要点

- 使用 `holder.setText()`, `holder.getTextView()`, `holder.getImageView()` 等方法

- 使用 `mDatas` 访问数据列表

- 设置数据：`holder.setText(R.id.tv_name, entity.getName());`

- **点击监听设置规范**：
  - `BaseRecylerAdapter` 基类已经自带 item 点击监听功能，通过 `setOnItemClickLitener()` 方法设置，通过 `setOnLongItemClickLitener()` 方法设置长按点击事件
  - 在`activity`或者`Fragment`的`bindEvent()`方法中设置`adapter.setOnItemClickLitener`
  - **不要为整个 item 定义自定义的监听接口**（如 `OnItemClickListener`），直接使用基类的 `setOnItemClickLitener()` 方法
  - **如果 item 内部有多个按钮**（如删除、编辑按钮），**需要为这些按钮定义专门的监听接口**
  - **点击监听必须在外部设置**（如 Activity、Fragment、Popup 中），**不要在 Adapter 构造函数中设置**
  - 如果只是简单的 item 点击，直接使用基类的 `setOnItemClickLitener()` 方法
  - 如果需要额外的业务逻辑（如选中状态更新、特殊处理等），可以在外部设置监听，然后调用 Adapter 提供的方法处理 UI 状态
  - 基类监听回调参数：`(View view, int position, T data)`，其中 `data` 是泛型数据对象，需要强制转换为具体类型
  - 示例：
  
```java
// ✅ 正确：在外部（Activity/Fragment/Popup）设置 item 点击监听
adapter = new XxxAdapter(context, list, R.layout.item_xxx);
adapter.setOnItemClickLitener((view, position, data) -> {
    XxxEntity entity = (XxxEntity) data; // 强制转换为具体类型
    // 处理业务逻辑
    handleItemClick(entity, position);
});
//// ✅ 正确：在外部（Activity/Fragment/Popup）设置 item 长按监听
adapter.setOnLongItemClickLitener((view, i) -> {  
    delete(adapter.getItem(i));  
});

// ✅ 正确：item 内部有多个按钮时，定义专门的监听接口
public class XxxAdapter extends BaseRecylerAdapter<XxxEntity> {
    private OnDeleteClickListener onDeleteClickListener;
    private OnEditClickListener onEditClickListener;
    //或者使用ButtonClickListener，给任意按钮设置点击事件,内部已经实现ButtonClickListener类，无需重复创建
    private ButtonClickListener<XxxEntity> buttonClickListener;  
  
	public void setButtonClickListener(ButtonClickListener<XxxEntity> buttonClickListener) {  
    this.buttonClickListener = buttonClickListener;  
}
    
    public interface OnDeleteClickListener {
        void onDeleteClick(XxxEntity entity, int position);
    }
    
    public interface OnEditClickListener {
        void onEditClick(XxxEntity entity, int position);
    }
    
    public void setOnDeleteClickListener(OnDeleteClickListener listener) {
        this.onDeleteClickListener = listener;
    }
    
    public void setOnEditClickListener(OnEditClickListener listener) {
        this.onEditClickListener = listener;
    }
    
    @Override
    public void convert(MyRecylerViewHolder holder, int position) {
        XxxEntity entity = mDatas.get(position);
        
        // 设置删除按钮监听
        holder.getView(R.id.btn_delete).setOnClickListener(v -> {
            if (onDeleteClickListener != null) {
                onDeleteClickListener.onDeleteClick(entity, position);
            }
        });
        
        // 设置编辑按钮监听
        holder.getView(R.id.btn_edit).setOnClickListener(v -> {
            if (onEditClickListener != null) {
                onEditClickListener.onEditClick(entity, position);
            }
        });
        
        //任意按钮设置点击事件
        holder.getView(R.id.btn).setOnClickListener(v -> {  
    if (buttonClickListener != null) {  
        buttonClickListener.onButtonClick(v, position, mDatas.get(position));  
    }  
});
    }
}

// ❌ 错误：不要在 Adapter 构造函数中设置监听
public XxxAdapter(Context context, List<XxxEntity> list, int layoutId) {
    super(context, list, layoutId);
    setOnItemClickLitener(...); // 不要这样做，应该在外部设置
}

// ❌ 错误：不要为整个 item 定义自定义的监听接口（基类已有）
public interface OnItemClickListener {
    void onItemClick(XxxEntity entity, int position);
}
private OnItemClickListener onItemClickListener; // 不要这样做，使用基类的监听
```

- 使用接口回调处理额外的点击事件，不要直接在 Adapter 中处理业务逻辑

# Entity 开发规范

## Room 数据库实体

```java
@Entity
public class XxxEntity implements Serializable {
    
    @PrimaryKey(autoGenerate = true)
    private long id;
    
    private String name;
    private long createTime;  // 创建时间，格式为时间戳
    
    // getter/setter 方法
    public long getId() { return id; }
    public void setId(long id) { this.id = id; }
    
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
    
    // ... 其他 getter/setter
}
```

## Entity 规范

- 使用 `@Entity` 注解
- 使用 `@PrimaryKey(autoGenerate = true)` 标记自增主键
- 实现 `Serializable` 接口
- 字段使用 `private` 修饰，提供 getter/setter 方法
- 时间字段使用 `long` 类型，格式为时间戳

## 非 Room 实体类规范

对于不需要存储到数据库的普通实体类，遵循以下规范：

- 实现 `Serializable` 接口（便于传递）
- 字段使用 `private` 修饰，提供 getter/setter 方法
- 空值处理由调用方（如 Adapter、UI 层）按需处理

```java
public class XxxEntity implements Serializable {
    private long id;
    private String name;
    
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
}
```

# Dao 开发规范

## Room DAO 接口

```java

@Dao
public interface XxxDao {
    
    @Insert(onConflict = OnConflictStrategy.REPLACE)  
	void insert(XxxEntity... beans);  
  
	@Insert(onConflict = OnConflictStrategy.REPLACE)  
	void insert(List<XxxEntity> list);
    
    @Insert(onConflict = OnConflictStrategy.REPLACE)
    void insertOrReplace(XxxEntity... entities);
    
    @Query("SELECT * FROM XxxEntity ORDER BY id DESC")
    List<XxxEntity> queryAll();
    
    @Query("SELECT COUNT(*) FROM XxxEntity")  
	int queryCount();  
  
	@Query("SELECT * FROM XxxEntity WHERE id = :id")  
	XxxEntity queryById(long id);
    
    @Update
    void update(XxxEntity... entities);
    
    @Delete
    void delete(XxxEntity... entities);
}
```

## Dao 使用

- 使用 `DatabaseManager.getInstance(context).getXxxDao()` 获取 Dao 实例

- 查询方法使用 `@Query` 注解

- 插入使用 `insert` 方法，使用 `OnConflictStrategy.REPLACE`

- 查询结果按 `id DESC` 排序（最新的在前）

# Room 数据库配置规范

## 开发阶段说明

当前项目处于开发阶段，无需考虑数据库迁移问题。若数据结构变更，可直接卸载 App 清除旧数据后重新安装，简化开发流程。

## 依赖配置

在模块的 build.gradle 文件中，添加 Room 数据库相关依赖，确保版本与项目中 androidApi 配置一致：

```groovy

// Room 数据库核心依赖
implementation androidApi.library.room
// 注解处理器
annotationProcessor androidApi.library.roomprocessor
// 支持 RxJava3 集成
implementation androidApi.library.roomRxjava3
```

## DatabaseManager 实现

数据库管理类采用单例模式设计，统一提供数据库实例访问入口，便于全局调用。需注意在实际使用时，将泛型占位符及数据库名称替换为项目真实信息。

```java

@Database(entities = {XxxEntity.class, YyyEntity.class}, version = 1, exportSchema = false)
public abstract class DatabaseManager extends RoomDatabase {

    // 数据库名称，可根据业务需求修改
    public static final String DB_NAME = "data.db";

    // 单例实例，使用 volatile 保证多线程可见性
    private volatile static DatabaseManager instance;

    /**
     * 获取数据库单例实例
     * @param context 上下文，建议使用 Application 上下文避免内存泄漏
     * @return 数据库实例
     */
    public static DatabaseManager getInstance(Context context) {
        if (instance == null) {
            synchronized (DatabaseManager.class) {
                if (instance == null) {
                    instance = DatabaseManager.create(context);
                }
            }
        }
        return instance;
    }

    /**
     * 创建数据库实例
     * 注意：不要使用 allowMainThreadQueries()，应使用 RxJava3 进行异步操作
     */
    private static DatabaseManager create(final Context context) {
        return Room.databaseBuilder(
                context.getApplicationContext(), // 使用应用上下文，防止内存泄漏
                DatabaseManager.class,
                DB_NAME)
                .allowMainThreadQueries()
                .build();
    }
    
    // 必须为每个 Dao 接口提供抽象的 getter 方法
    public abstract XxxDao getXxxDao();
    public abstract YyyDao getYyyDao();

}
```

## 使用要点

- `@Database` 注解中 `entities` 参数需包含所有 Room 实体类，用逗号分隔

- **允许使用 `allowMainThreadQueries()`**，同时也使用 **RxJava3** 进行异步数据库操作（项目已集成 RxJava3）

- 获取实例时建议传入 `Application` 上下文，防止因 Activity 上下文销毁导致的内存泄漏

- 每个 Dao 接口都需在 DatabaseManager 中定义对应的抽象 getter 方法，如 `getXxxDao()`

## 数据库与异步操作规范（唯一标准）

### 核心原则

1. **Dao 接口只返回普通类型**（`List`、`void` 等），保持纯粹，不依赖 RxJava
2. **UI 层使用 `Observable.create` 包裹 Dao 操作**
3. **严禁使用主线程查库**，必须使用 `subscribeOn(Schedulers.io())`

### Dao 接口定义

```java
@Dao
public interface XxxDao {
    // 查询返回 List（普通类型，不需要 Observable）
    @Query("SELECT * FROM XxxEntity ORDER BY id DESC")
    List<XxxEntity> queryAll();
    
    // 查询单个值返回 Double
    @Query("SELECT SUM(amount2) FROM XxxEntity WHERE month = :month AND type = :type")
    Double getTotalAmountByMonth(String type, String month);
    
    // 插入返回 void
    @Insert(onConflict = OnConflictStrategy.REPLACE)
    void insert(XxxEntity entity);
    
    // 更新返回 void
    @Update
    void update(XxxEntity entity);
    
    // 删除返回 void
    @Query("DELETE FROM XxxEntity WHERE id = :id")
    void deleteById(int id);
}
```

### 查库标准写法（复制即用）

```java
import io.reactivex.rxjava3.core.Observable;
import io.reactivex.rxjava3.core.ObservableOnSubscribe;
import io.reactivex.rxjava3.schedulers.Schedulers;
import io.reactivex.rxjava3.android.schedulers.AndroidSchedulers;
import java.util.ArrayList;

private void getData() {  
    Observable.create((ObservableOnSubscribe<List<XxxEntity>>) emitter -> {  
                List<XxxEntity> list = DatabaseManager.getInstance(this).getXxxDao().queryAll();
        emitter.onNext(list);
            })  
            .subscribeOn(Schedulers.io())  
            .observeOn(AndroidSchedulers.mainThread())  
            .subscribe(new SimpleObserver<List<XxxEntity>>() {  
                @Override  
                public void onNext(List<XxxEntity> list) {
	                // 2. 【主线程】更新 UI
                    adapter.addAllAndClear(list)
                    binding.tvDataEmpty.setVisibility(adapter.getItemCount() > 0 ? View.GONE : View.VISIBLE);  
                }  
  
            });  
  
}
```

### 保存数据标准写法（含验证）

```java
private void saveData() {
    // 1. 先进行表单验证（同步逻辑）
    String name = binding.etName.getText().toString().trim();
    if (TextUtils.isEmpty(name)) {
        ToastUtils.showShort("名称不能为空");
        return;
    }
    
    final XxxEntity entity = new XxxEntity();
    entity.setName(name);

    // 2. 验证通过后，异步保存
    Observable.create((ObservableOnSubscribe<XxxEntity>) emitter -> {  
            // 【子线程】写入
            DatabaseManager.getInstance(getApplicationContext()).getXxxDao().insert(entity);  
        })  
        .subscribeOn(Schedulers.io())  
        .observeOn(AndroidSchedulers.mainThread())  
        .subscribe(new SimpleObserver<XxxEntity>() {  
            @Override  
            public void onNext(XxxEntity entity) {  
	            // 【主线程】反馈
                ToastUtils.showShort("保存成功");  
            }  
  
            @Override  
            public void onError(Throwable e) {  
                super.onError(e);  
                ToastUtils.showShort("保存失败");  
            }  
        });
    
```

### 删除数据标准写法

```java
private void deleteData(XxxEntity entity, int position) {
    Observable.create((ObservableOnSubscribe<XxxEntity>) emitter -> {  
            // 【子线程】删除
            DatabaseManager.getInstance(getApplicationContext()).getXxxDao().delete(entity);  
        })  
    .subscribeOn(Schedulers.io())
    .observeOn(AndroidSchedulers.mainThread())
    .subscribe(new SimpleObserver<XxxEntity>() {  
            @Override  
            public void onNext(XxxEntity entity) {  
	            // 【主线程】更新UI
                ToastUtils.showShort("删除成功");  
            }  
  
            @Override  
            public void onError(Throwable e) {  
                super.onError(e);  
                ToastUtils.showShort("删除失败");  
            }  
        });
}
```

### 注意事项

- **Dao 接口保持纯粹**：只返回普通类型（`List<T>`、`Double`、`void` 等），不返回 `Observable`/`Single`/`Completable`
- **线程切换**：`subscribeOn(Schedulers.io())` 指定在 IO 线程执行数据库操作，`observeOn(AndroidSchedulers.mainThread())` 指定在主线程观察结果
- **表单验证**：保存数据前先进行同步验证，验证通过后再异步保存，提升用户体验


# 工具类使用规范

## 常用工具类

- **Toast 提示**：`ToastUtils.showShort("消息")`

- **确认对话框**：

```java
// 方式一：使用 DialogUtil 快捷方法，默认使用该方式，执行删除操作前必须提示该弹窗
DialogUtil.showConfirmRreceiptDialog(context, "标题", "确认要执行此操作吗？", 
    new ConfirmDialog.OnDialogClickListener() {
        @Override
        public void confirm() {
            // 点击确认按钮
        }
        
        @Override
        public void cancel() {
            // 点击取消按钮
        }
    });

// 方式二：使用 ConfirmDialog.Builder（更灵活）
new ConfirmDialog.Builder(context)
    .setTitleName("标题")
    .setMessage("确认要删除吗？")
    .setConfirmBtn("确定")
    .setCancelBtn("取消")
    .setOnDialogClickListener(new ConfirmDialog.OnDialogClickListener() {
        @Override
        public void confirm() {
            // 点击确认
        }
        
        @Override
        public void cancel() {
            // 点击取消
        }
    })
    .create(true)  // true: 点击外部可关闭
    .show();
```

**需要导入**：
```java
import com.viterbi.common.widget.dialog.DialogUtil;
import com.viterbi.common.widget.dialog.ConfirmDialog;
```

- **时间格式化**：
  - `VTBTimeUtils.formatDateTime(System.currentTimeMillis())` - 格式化时间戳
  - `VTBTimeUtils.getCurrentTime()` - 获取当前时间字符串（格式：yyyy-MM-dd）
  - `VTBTimeUtils.strToDate(String dateStr)` - 字符串转 Date 对象

- **尺寸转换**：`SizeUtils.dp2px(12)`

## 数据库访问

**注意：** 以下代码仅用于说明 Dao 接口的使用，实际使用时必须通过 `Observable.create()` 进行异步操作，详见"数据库与异步操作规范"章节。

```java
// 获取 Dao
XxxDao dao = DatabaseManager.getInstance(context).getXxxDao();

// 查询所有（必须在 Observable.create 中使用）
List<XxxEntity> list = dao.queryAll();

// 插入或更新（必须在 Observable.create 中使用）
dao.insertOrReplace(entity);

// 删除（必须在 Observable.create 中使用）
dao.delete(entity);
```

# 网络请求规范

## Presenter 模式使用

如果需要请求网络接口，Activity 需要：

1. 继承 BaseActivity<Binding, MainContract.Presenter>`

2. 实现 `MainContract.View` 接口

3. 在 `initView()` 中初始化 presenter 并调用接口方法

## 标准结构

```java

public class XxxListActivity extends BaseActivity<ActivityXxxListBinding, MainContract.Presenter> implements MainContract.View {
    private XxxAdapter adapter;
    private String dataUrl;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setDataBindingLayout(R.layout.activity_xxx_list);
    }

    @Override
    public void initView() {
	    // 初始化工具栏
        binding.include.setTitleStr("标题");
    
        // 获取参数
        dataUrl = getIntent().getStringExtra("dataUrl");
        
        // 初始化 RecyclerView
        adapter = new XxxAdapter(this, list, R.layout.item_xxx);
		binding.rv.setLayoutManager(new LinearLayoutManager(this));
		binding.rv.addItemDecoration(new SimplePaddingDecoration(this, SizeUtils.dp2px(0)));  // 设置间距
		binding.rv.setAdapter(adapter);
        
        // 初始化 presenter 并请求数据
        presenter = new MainPresenter(this);
        presenter.queryJson(dataUrl);
    }

    @Override
    public void bindEvent() {
        binding.setOnClickListener(this::onClickCallback);
    }
    
    @Override  
	public void onClickCallback(View view) {  
	    switch (view.getId()) {  
	        case R.id.iv_title_back:  
            finish();  
            break;  
    }  
}

    // 实现 View 接口方法：请求成功回调
    @Override
    public void queryJsonSuccess(String requestUrl, String jsonStr) {
        List<XxxEntity> list = new Gson().fromJson(jsonStr, new TypeToken<List<XxxEntity>>() {
        }.getType());
        showList(list);
    }

    // 实现 View 接口方法：隐藏加载对话框
    @Override
    public void hideLoading() {
        hideLoadingDialog();
    }

    // 显示数据列表
    private void showList(List<XxxEntity> list) {
	    adapter.addAllAndClear(list);
	    binding.tvEmpty.setVisibility(adapter.getItemCount() > 0 ? View.GONE : View.VISIBLE);

    }
    
    //或者可以直接使用存储在数据库中的数据，详细用法参照"查库标准写法"
}
```

## 网络请求要点

- 在 `initView()` 中初始化 `presenter = new MainPresenter(this)`

- 调用 `presenter.queryJson(dataUrl)` 发起请求

- 实现 `queryJsonSuccess()` 方法处理请求成功的数据

- 实现 `hideLoading()` 方法隐藏加载对话框

- 使用 `adapter.addAllAndClear(list)` 更新列表数据

# 代码风格规范

## 命名规范

- **Activity**：`XxxActivity` 或 `XxxListActivity`、`AddXxxActivity`

- **Fragment**：`XxxFragment`

- **Adapter**：`XxxListAdapter` 或 `XxxAdapter`

- **Entity**：`XxxEntity`

- **Dao**：`XxxDao`

- **布局文件**：Activity 使用 `activity_xxx`，Fragment 使用 `fra_xxx`，RecyclerView item 使用 `item_xxx`， `item_xxx`存放在`res\layout\list\layout\`路径下

## 变量命名

- 使用驼峰命名法

- Adapter 变量名：`adapter`

- 数据列表：`list` 或 `mDatas`（Adapter 内部）

- Binding 变量：`binding`

- Context：`context` 或 `mContext`（Activity 中）

## 代码组织

- 成员变量声明在类顶部

- `onCreate()` -> `initView()` -> `bindEvent()` -> `onClickCallback()` 顺序

- 私有方法放在公共方法后面

- 相关功能的方法放在一起

## 点击事件处理规范

- 在 `onClickCallback()` 方法中处理点击事件
- if-else 和 switch-case 均可使用，根据实际情况选择

```java
@Override
public void onClickCallback(View view) {
    int id = view.getId();
    if (id == R.id.iv_title_back) {
        finish();
    } else if (id == R.id.tv_title_right) {
        saveData();
    }
}
```

## 包名和导入规范

- **优先使用 import 导入类，直接使用类名**

- 只有在存在同名类需要区分时才使用全包名

- 示例：

```java
// ✅ 正确：使用 import 导入后直接使用类名
import com.xxx.project.widget.pop.XxxSelectPopup;

XxxSelectPopup popup = new XxxSelectPopup(this, ...);

// ❌ 错误：不必要的全包名
com.xxx.project.widget.pop.XxxSelectPopup popup = 
    new com.xxx.project.widget.pop.XxxSelectPopup(this, ...);

// ✅ 正确：存在同名类时才使用全包名区分
import com.xxx.project.widget.pop.XxxSelectPopup;
import com.other.package.XxxSelectPopup; // 同名类

// 需要区分时使用全包名
com.xxx.project.widget.pop.XxxSelectPopup popup1 = ...;
com.other.package.XxxSelectPopup popup2 = ...;
```

## 空值处理

在 UI 层（Adapter、Activity 等）按需进行空值处理：

- 字符串展示：`str == null ? "" : str`
- 视图操作：`if (view != null) { ... }`
- 列表操作：`if (list != null) list.addAll(data)`

## 注释规范

- 使用中文注释

- 关键业务逻辑必须添加注释

- 复杂逻辑添加说明性注释

# 布局文件规范

## 布局文件组织

- Activity 布局：`activity_xxx.xml`，需要在Activity的`bindEvent()`方法中设置`binding.setOnClickListener(this::onClickCallback)`，在kotlin语法中也是使用该语法设置点击事件监听，不要使用`binding.onClickListener = this::onClickCallback`，标题栏为了保证风格统一使用`layout="@layout/layout_title_bar" `
```
<?xml version="1.0" encoding="utf-8"?>  
<layout xmlns:android="http://schemas.android.com/apk/res/android"  
    xmlns:app="http://schemas.android.com/apk/res-auto">  
  
    <data>  
  
        <variable  
            name="onClickListener"  
            type="android.view.View.OnClickListener" />  
  
    </data>  
  
    <androidx.constraintlayout.widget.ConstraintLayout  
        android:layout_width="match_parent"  
        android:layout_height="match_parent"  
        android:background="@color/base_bg_color">  
  
        <include  
            android:id="@+id/include"  
            layout="@layout/layout_title_bar"  
            android:onClickListener="@{onClickListener}"  
            app:layout_constraintEnd_toEndOf="parent"
            app:layout_constraintStart_toStartOf="parent"
            app:layout_constraintTop_toTopOf="parent" />  
  
        <androidx.recyclerview.widget.RecyclerView  
            android:id="@+id/rv"  
            android:layout_width="match_parent"  
            android:layout_height="0dp"  
            app:layout_constraintBottom_toBottomOf="parent"  
            app:layout_constraintTop_toBottomOf="@+id/include" />  
  
    </androidx.constraintlayout.widget.ConstraintLayout>  
  
</layout>
```
- 给任意view设置点击事件
```
<TextView  
    android:id="@+id/tv"  
    android:wrap_content="wrap_content"  
    android:wrap_content="wrap_content"  
    android:onClickListener="@{onClickListener}"  
```
- Fragment 布局：`fra_xxx.xml`

- RecyclerView item 布局：`item_xxx.xml`，需要存放在`res\layout\list\layout\`路径下

- Dialog 布局：`dialog_xxx.xml`，需要存放在`res\layout\dialog\layout\`路径下

- View 布局：`view_xxx.xml`，，需要存放在`res\layout\view\layout\`路径下

- Popup 布局：`popup_xxx.xml`，需要存放在`res\layout\dialog\layout\`路径下

- xml布局中`<layout>`下的根布局尽量使用ConstraintLayout


## XML 编码规范

Android XML 编码统一遵循简洁复用原则：资源 ID 使用带控件类型前缀的小写下划线命名；图标以 `ic_` 开头，避免页面名和 `icon` 等冗余词，大插图以 `_art` 区分；除非设计明确要求，`ImageView` 和 `TextView` 均使用 `wrap_content`，文本不指定字体及字体边距；容器不随意固定高度或设置统一外边距，由内部控件控制间距；相同样式优先复用已有 drawable，禁止重复创建资源；圆角背景需要使用对应的drawable资源，要根据圆角尺寸使用对应的drawable资源，shape_bg_white_xx固定为白色背景，使用backgroundTint属性设置为其他颜色

## DataBinding 使用

- 所有布局文件根标签使用 `<layout>` 包裹

- 使用 `binding` 变量访问所有视图

- 在 Activity/Fragment 中通过 `binding.xxx` 访问视图



## 圆角图片使用

**重要**：Glide 的 `RoundedCorners` 变换已不生效，**必须使用 RoundedImageView 库实现圆角图片**。

### 依赖配置

```groovy
api 'com.makeramen:roundedimageview:2.3.0'
```

### XML 布局中使用

```xml
<com.makeramen.roundedimageview.RoundedImageView
    android:id="@+id/iv_cover"
    android:layout_width="match_parent"
    android:layout_height="120dp"
    android:scaleType="centerCrop"
    app:riv_corner_radius="8dp"
    app:riv_oval="false" />
```

### 常用属性

- `app:riv_corner_radius` - 统一设置四个角的圆角半径
- `app:riv_corner_radius_top_left` - 单独设置左上角
- `app:riv_corner_radius_top_right` - 单独设置右上角
- `app:riv_corner_radius_bottom_left` - 单独设置左下角
- `app:riv_corner_radius_bottom_right` - 单独设置右下角
- `app:riv_oval="true"` - 设置为圆形图片
- `app:riv_border_width` - 边框宽度
- `app:riv_border_color` - 边框颜色

### 配合 Glide 加载网络图片

```java
// 直接加载，RoundedImageView 会自动处理圆角
Glide.with(context)
    .load(imageUrl)
    .into(holder.getView(R.id.iv_cover));
```

### 错误示例（不要使用）

```java
// ❌ 错误：Glide 的 RoundedCorners 不生效
RequestOptions options = new RequestOptions()
    .transform(new RoundedCorners(24));
Glide.with(context)
    .load(imageUrl)
    .apply(options)
    .into(imageView);
```

## RadioButton/RadioGroup 使用

- 使用 `RadioGroup` 包裹多个 `RadioButton` 实现单选

- 使用 `OnCheckedChangeListener` 监听选择变化

- 在监听器中判断 `isChecked` 为 `true` 时才处理逻辑

- 示例：

```java
binding.rbOption1.setOnCheckedChangeListener((buttonView, isChecked) -> {
    if (isChecked) {
        // 处理选中逻辑
        currentOption = OPTION_1;
        updateUI();
    }
});
```

# 注意事项

1. **`findViewById()`** **不要使用 **，必须使用 DataBinding/ViewBinding

2. **`onCreate()`** **`setDataBindingLayout()`** **Activity 必须在  中调用 **

3. **`onSetLayoutId()`** **Fragment 必须实现  方法**

4. **`convert()`** **Adapter 必须实现  方法绑定数据**

5. **`BaseRecylerAdapter`** **基类自带点击监听，必须在外部设置，不要在 Adapter 构造函数中设置**

6. **`Serializable`** **Entity 必须实现  接口**

7. **在 UI 展示时按需进行空值检查**

8. **`adapter.addAllAndClear(list)`** **`mDatas`** **使用  更新数据，不要直接操作 **

9. **`LayoutManager`** **`ItemDecoration`** **RecyclerView 必须设置  和 **

10. **`DatabaseManager.getInstance()`** **使用  获取数据库实例**

11. **`VTBTimeUtils.formatDateTime()`** **时间字段统一使用  格式化**

12. **新写的 Activity 一定要在 AndroidManifest.xml 中注册，注册示例```<activity  
    android:name=".XxxActivity"  
    android:screenOrientation="portrait"  
    tools:ignore="DiscouragedApi,LockedOrientationActivity" />```
    **

13. **数据库开发阶段可通过卸载 App 清除旧数据，无需处理迁移**

14. **`allowMainThreadQueries()`** **数据库操作允许使用 **

15. **优先使用 import 导入类，直接使用类名，避免不必要的全包名，只有在存在同名类需要区分时才使用全包名**

16. **数据保存前必须进行表单验证，使用 `TextUtils.isEmpty()` 检查必填字段**

17. **数据库操作必须使用 try-catch 捕获异常，并给用户友好的错误提示**

18. **自定义弹窗必须继承 `BottomPopupView`，实现 `getImplLayoutId()` 和 `onCreate()` 方法**

19. **使用 `RadioButton` 和 `RadioGroup` 实现单选时，在 `OnCheckedChangeListener` 中判断 `isChecked` 为 `true` 才处理逻辑**

20. **圆角图片必须使用 `RoundedImageView` 库，不要使用 Glide 的 `RoundedCorners`（已不生效）**

21. **Toast使用`com.blankj.utilcode.util.ToastUtils`下面的`ToastUtils.showShort("")`**

22. **在Activity或Fragment中无参数跳转使用`skipAct(XxxActivity.class);`，携带参数跳转使用``` Intent intent =new  Intent(this, XxxActivity.class)  
intent.putExtra("data", entity)  
startActivity(intent) ```
    **

# 弹窗开发规范

## XPopup 使用

项目使用 `XPopup` 库实现自定义弹窗，所有自定义弹窗必须继承 `BottomPopupView`：

```java
public class XxxSelectPopup extends BottomPopupView {
    private XxxBinding binding;
    private OnXxxSelectListener listener;
    
    public interface OnXxxSelectListener {
        void onXxxSelected(XxxEntity entity);
    }
    
    public XxxSelectPopup(Context context, OnXxxSelectListener listener) {
        super(context);
        this.listener = listener;
    }
    
    @Override
    protected int getImplLayoutId() {
        return R.layout.popup_xxx;
    }
    
    @Override
    protected void onCreate() {
        super.onCreate();
        binding = XxxBinding.bind(getPopupImplView());
        // 初始化视图和逻辑
    }
}
```

## 显示弹窗

使用 `XPopup.Builder` 显示自定义弹窗：

```java
XxxSelectPopup popup = new XxxSelectPopup(this, new XxxSelectPopup.OnXxxSelectListener() {
    @Override
    public void onXxxSelected(XxxEntity entity) {
        // 处理选择结果
    }
});
new XPopup.Builder(this)
    .asCustom(popup)
    .show();
```

## 日期时间选择器

**重要说明**：实际开发时以 UI 设计图为准，文档仅作为技术参考。项目已封装统一的日期时间选择器，优先使用项目封装类。

### 项目统一使用：DateTimeSelectPopup ⭐

项目已封装 `DateTimeSelectPopup` 类，统一处理月份和日期选择，支持自定义 UI 样式。

**使用方式**：

```java
// 选择月份
DateTimeSelectPopup popup = new DateTimeSelectPopup(context, DateTimeSelectPopup.TYPE_MONTH, currentMonth, month -> {
    // month: "yyyy-MM" 格式
    // 处理选择结果
});
new XPopup.Builder(context).asCustom(popup).show();

// 选择日期
DateTimeSelectPopup popup = new DateTimeSelectPopup(context, DateTimeSelectPopup.TYPE_DATE, currentDate, date -> {
    // date: "yyyy-MM-dd" 格式
    // 处理选择结果
});
new XPopup.Builder(context).asCustom(popup).show();
```

**特点**：
- 统一的 UI 样式（粉色圆角选中背景）
- 支持月份选择（TYPE_MONTH）和日期选择（TYPE_DATE）
- 样式配置在 XML 中，代码只处理动态逻辑
- 自动处理时间格式化和空值

**布局文件**：
- 月份选择：`popup_select_month.xml`
- 日期选择：`popup_select_date.xml`

**自定义 UI**：
- 通过修改 XML 布局文件中的 `DateTimePicker` 属性来调整样式
- 使用 `app:dt_*` 属性在 XML 中配置颜色、字体等
- 如需自定义选中背景，修改 `view_highlight_bg` 的 drawable

### 其他选择器（参考）

根据 UI 设计需求，可选择使用以下选择器库（项目本身未默认导入，需要时手动添加依赖）：

1. **DateTimePicker 库**：包含以下组件
   - `DateTimePicker`：核心选择器组件，可在 XML 中直接使用
   - `CardDatePickerDialog`：对话框形式的日期选择器，开箱即用
   - `CardWeekPickerDialog`：周选择器

2. **calendarview 库**：日历选择器
   - 依赖：`implementation 'com.haibin:calendarview:3.7.1'`

如需使用这些选择器，请先添加对应依赖，然后参考库文档并根据 UI 设计图实现。

# 数据验证规范

## 表单验证

在保存数据前必须进行验证：

```java
private void saveData() {
    // 1. 验证必填字段
    String fieldValue = binding.etField.getText().toString().trim();
    if (TextUtils.isEmpty(fieldValue)) {
        ToastUtils.showShort("请输入必填字段");
        return;
    }
    
    // 2. 验证数据类型（如数字类型）
    float numberValue = 0;
    try {
        numberValue = Float.parseFloat(fieldValue);
        if (numberValue <= 0) {
            ToastUtils.showShort("数值必须大于0");
            return;
        }
    } catch (NumberFormatException e) {
        ToastUtils.showShort("请输入有效的数值");
        return;
    }
    
    // 3. 根据业务逻辑验证不同字段
    if (CURRENT_TYPE.equals(someType)) {
        if (TextUtils.isEmpty(selectedItem)) {
            ToastUtils.showShort("请选择必选项");
            return;
        }
    }
    
    // 4. 保存数据
    saveToDatabase();
}
```

## 错误处理

数据库操作必须使用 try-catch 捕获异常：

```java
try {
    dao.insertOrReplace(entity);
    ToastUtils.showShort("保存成功");
    finish();
} catch (Exception e) {
    e.printStackTrace();
    ToastUtils.showShort("保存失败：" + e.getMessage());
}
```

# 依赖库

项目使用的主要依赖：

## 项目已有依赖（无需添加）

- DataBinding / ViewBinding

- RecyclerView

- Glide（图片加载）

- Retrofit + RxJava3（网络请求）

- Gson（JSON 解析）

- **[AndroidUtilCode](https://github.com/Blankj/AndroidUtilCode)**（强大易用的安卓工具类库，它合理地封装了安卓开发中常用的函数）

## 需要手动添加的依赖

- **Room 数据库**：
```groovy
implementation androidApi.library.room
annotationProcessor androidApi.library.roomprocessor
implementation androidApi.library.roomRxjava3
```

- **RoundedImageView（圆角图片）**：
```groovy
api 'com.makeramen:roundedimageview:2.3.0'
```

- **XPopup（弹窗库）**：
```groovy
implementation 'com.github.li-xiaojun:XPopup:2.10.0'
```

- **日历选择器**：
```groovy
implementation 'com.haibin:calendarview:3.7.1'
```

- **图片选择器**：
```groovy
implementation 'io.github.lucksiege:pictureselector:v3.11.2'
```

- **图片压缩**：
```groovy
implementation 'io.github.lucksiege:compress:v3.11.2'
```

# 包结构

```text

com.xxx.project_name/
├── ui/
│   ├── adapter/          # 所有 Adapter
│   ├── mime/             # 主要业务模块
│   │   ├── main/         # 主界面
	│── ── .../           # 其他业务模块
│   └── ...
├── widget/
│   └── pop/              # 自定义弹窗类（可选）
├── dao/                  # 所有 Dao 接口
├── entitys/              # 所有 Entity 实体类（Room 和非 Room）
├── utils/                # 工具类
└── common/               # 公共类（如基类、契约类等）
```