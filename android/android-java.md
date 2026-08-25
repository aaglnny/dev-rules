# Android Java 语言开发规范

## 适用范围

本规范只负责 Java 语言写法和 Java 实现示例。Android 项目的公共架构、组件职责、文件语言边界、Room、依赖、Manifest 和包结构统一读取 [android-project.md](android-project.md)；XML、Drawable 和资源规则统一读取 [android-xml.md](android-xml.md)。

使用本规范前必须先加载 `E:\rules\android\android-project.md`。涉及布局或资源时再加载 `E:\rules\android\android-xml.md`。

`dao`、`entitys` 包必须使用 Java，因此 Entity、Dao 和常用 `DatabaseManager` 的实现示例保留在本文件中。

## Java 完整示例索引

根据任务读取对应文件，不一次性加载全部示例：

- Activity：`E:\rules\android\examples\activity-java.md`
- Fragment：`E:\rules\android\examples\fragment-java.md`
- Adapter：`E:\rules\android\examples\adapter-java.md`
- Entity、Dao、DatabaseManager：`E:\rules\android\examples\room-java.md`
- RxJava3 数据库异步：`E:\rules\android\examples\rxjava-java.md`
- 自定义 Dialog：`E:\rules\android\examples\dialog-java.md`

完整示例的强制程度、可替换范围和真实项目核对顺序统一遵守 [android-project.md](android-project.md) 中的“完整示例使用规则”。

# Java 基础规则

## 版本与语法边界

- Java 版本以项目 Gradle 配置为准，不在普通业务需求中擅自升级。
- 优先使用当前项目已经稳定采用的 Java 写法，不为展示新语法改写旧代码。
- Java 与 Kotlin 互调时保持现有公开方法名、字段名和历史拼写不变。

## 字段与局部变量

- 成员字段声明在类顶部，默认使用 `private`。
- 不会重新赋值的字段按实际情况使用 `final`，不要机械地给所有局部变量补 `final`。

## 空值处理

- 根据数据真实来源进行空值判断，不为项目内部已保证非空的数据层层增加判断。
- 字符串展示可使用三元表达式转换为空字符串。
- 只有对象真实可能为空时才进行 `null` 判断。
- 列表为空与列表对象为 `null` 分开判断，不用异常代替正常分支。

```java
String displayName = entity.getName() == null ? "" : entity.getName();

if (entity == null) {
    return;
}
```

## Lambda、方法引用与匿名类

- 单方法接口优先使用 Lambda 或方法引用，以当前文件风格为准。
- 多方法接口使用匿名内部类。
- Lambda 逻辑较长时提取为有业务含义的方法。
- 已有项目接口能满足需求时不重复定义回调接口。

## 分支与方法组织

- 点击事件可使用 `if/else` 或 `switch`，根据分支数量和当前文件风格选择。
- 方法默认按生命周期方法、基类重写方法、接口回调、私有业务方法组织。
- 相关方法放在一起，不为绝对顺序移动无关旧代码。
- 私有方法放在公开方法和重写方法之后。

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

## 格式

- 以 Android Studio/Java 默认格式为基础，不顺手格式化整个旧文件。

# Android 组件 Java 示例

以下示例只说明如何用 Java 落实 [android-project.md](android-project.md) 的公共规则，不重新定义组件职责。

## Activity

```java
public class XxxActivity extends BaseActivity<ActivityXxxBinding, BasePresenter> {
    private XxxAdapter adapter;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setDataBindingLayout(R.layout.activity_xxx);
    }

    @Override
    public void initView() {
        binding.include.setTitleStr("标题");

        adapter = new XxxAdapter(this, null, R.layout.item_xxx);
        binding.rv.setLayoutManager(new LinearLayoutManager(this));
        binding.rv.addItemDecoration(
                new SimplePaddingDecoration(this, SizeUtils.dp2px(0)));
        binding.rv.setAdapter(adapter);

        getData();
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

    private void getData() {
        // 按项目数据库或网络规则加载数据
    }

    private void saveData() {
        // 先验证，再按项目异步规则保存
    }
}
```

网格列表只替换布局管理器和间距实现：

```java
binding.rv.setLayoutManager(new GridLayoutManager(this, 2));
binding.rv.addItemDecoration(
        new GridSpacesItemDecoration(2, SizeUtils.dp2px(8f), false));
```

## Fragment

```java
public class XxxFragment extends BaseFragment<FraXxxBinding, BasePresenter> {

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
        if (id == R.id.iv_title_back && getActivity() != null) {
            getActivity().finish();
        } else if (id == R.id.tv_title_right) {
            saveData();
        }
    }

    private void saveData() {
        // 保存业务数据
    }
}
```

带参数实例：

```java
private static final String ARG_ID = "arg_id";

public static XxxFragment newInstance(long id) {
    XxxFragment fragment = new XxxFragment();
    Bundle args = new Bundle();
    args.putLong(ARG_ID, id);
    fragment.setArguments(args);
    return fragment;
}
```

## Adapter

```java
public class XxxListAdapter extends BaseRecylerAdapter<XxxEntity> {
    private final Context context;

    public XxxListAdapter(Context context, List<XxxEntity> list, int layoutId) {
        super(context, list, layoutId);
        this.context = context;
    }

    @Override
    public void convert(MyRecylerViewHolder holder, int position) {
        XxxEntity entity = mDatas.get(position);
        String name = entity.getName() == null ? "" : entity.getName();

        holder.setText(R.id.tv_name, name);
        Glide.with(context)
                .load(entity.getImageUrl())
                .into(holder.getImageView(R.id.iv_cover));
    }
}
```

设置基类 Item 监听：

Item 监听统一放在页面的 `bindEvent()` 中，Adapter 初始化和 RecyclerView 绑定放在 `initView()` 中。

```java
adapter.setOnItemClickLitener((view, position, data) -> {
    XxxEntity entity = (XxxEntity) data;
    handleItemClick(entity, position);
});

adapter.setOnLongItemClickLitener((view, position) -> {
    delete(adapter.getItem(position));
});
```

Item 内按钮回调：

```java
public class XxxAdapter extends BaseRecylerAdapter<XxxEntity> {
    private ButtonClickListener<XxxEntity> buttonClickListener;

    public XxxAdapter(Context context, List<XxxEntity> list, int layoutId) {
        super(context, list, layoutId);
    }

    public void setButtonClickListener(
            ButtonClickListener<XxxEntity> buttonClickListener) {
        this.buttonClickListener = buttonClickListener;
    }

    @Override
    public void convert(MyRecylerViewHolder holder, int position) {
        XxxEntity entity = mDatas.get(position);

        holder.getView(R.id.btn_action).setOnClickListener(view -> {
            if (buttonClickListener != null) {
                buttonClickListener.onButtonClick(view, position, entity);
            }
        });
    }
}
```

需要语义明确的专用回调时：

```java
public interface OnDeleteClickListener {
    void onDeleteClick(XxxEntity entity, int position);
}
```

# Entity 与 Dao Java 示例

## Room Entity

```java
@Entity
public class XxxEntity implements Serializable {
    @PrimaryKey(autoGenerate = true)
    private long id;

    private String name;
    private long createTime;

    public long getId() {
        return id;
    }

    public void setId(long id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public long getCreateTime() {
        return createTime;
    }

    public void setCreateTime(long createTime) {
        this.createTime = createTime;
    }
}
```

该示例对应 [android-project.md](android-project.md) 中的 Entity 公共约束，约束本身以项目规范为准。

## 非 Room Entity

```java
public class XxxEntity implements Serializable {
    private long id;
    private String name;

    public long getId() {
        return id;
    }

    public void setId(long id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}
```

## Room Dao

```java
@Dao
public interface XxxDao {

    @Insert(onConflict = OnConflictStrategy.REPLACE)
    void insert(XxxEntity... entities);

    @Insert(onConflict = OnConflictStrategy.REPLACE)
    void insert(List<XxxEntity> list);

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

## DatabaseManager

```java
@Database(
        entities = {XxxEntity.class, YyyEntity.class},
        version = 1,
        exportSchema = false)
public abstract class DatabaseManager extends RoomDatabase {
    public static final String DB_NAME = "data.db";

    private static volatile DatabaseManager instance;

    public static DatabaseManager getInstance(Context context) {
        if (instance == null) {
            synchronized (DatabaseManager.class) {
                if (instance == null) {
                    instance = create(context);
                }
            }
        }
        return instance;
    }

    private static DatabaseManager create(Context context) {
        return Room.databaseBuilder(
                        context.getApplicationContext(),
                        DatabaseManager.class,
                        DB_NAME)
                .allowMainThreadQueries()
                .build();
    }

    public abstract XxxDao getXxxDao();

    public abstract YyyDao getYyyDao();
}
```

# Room 与 RxJava3 Java 示例

## 查询

```java
private void getData() {
    Observable.create((ObservableOnSubscribe<List<XxxEntity>>) emitter -> {
                try {
                    List<XxxEntity> list = DatabaseManager
                            .getInstance(getApplicationContext())
                            .getXxxDao()
                            .queryAll();
                    emitter.onNext(list);
                    emitter.onComplete();
                } catch (Exception e) {
                    emitter.tryOnError(e);
                }
            })
            .subscribeOn(Schedulers.io())
            .observeOn(AndroidSchedulers.mainThread())
            .subscribe(new SimpleObserver<List<XxxEntity>>() {
                @Override
                public void onNext(List<XxxEntity> list) {
                    adapter.addAllAndClear(list);
                    binding.tvEmpty.setVisibility(
                            adapter.getItemCount() > 0 ? View.GONE : View.VISIBLE);
                }

                @Override
                public void onError(Throwable e) {
                    super.onError(e);
                    ToastUtils.showShort("数据加载失败");
                }
            });
}
```

## 保存

```java
private void saveData() {
    String name = binding.etName.getText().toString().trim();
    if (TextUtils.isEmpty(name)) {
        ToastUtils.showShort("名称不能为空");
        return;
    }

    XxxEntity entity = new XxxEntity();
    entity.setName(name);

    Observable.create((ObservableOnSubscribe<Boolean>) emitter -> {
                try {
                    DatabaseManager
                            .getInstance(getApplicationContext())
                            .getXxxDao()
                            .insert(entity);
                    emitter.onNext(Boolean.TRUE);
                    emitter.onComplete();
                } catch (Exception e) {
                    emitter.tryOnError(e);
                }
            })
            .subscribeOn(Schedulers.io())
            .observeOn(AndroidSchedulers.mainThread())
            .subscribe(new SimpleObserver<Boolean>() {
                @Override
                public void onNext(Boolean success) {
                    ToastUtils.showShort("保存成功");
                    finish();
                }

                @Override
                public void onError(Throwable e) {
                    super.onError(e);
                    ToastUtils.showShort("保存失败");
                }
            });
}
```

## 删除

```java
private void deleteData(XxxEntity entity) {
    Observable.create((ObservableOnSubscribe<Boolean>) emitter -> {
                try {
                    DatabaseManager
                            .getInstance(getApplicationContext())
                            .getXxxDao()
                            .delete(entity);
                    emitter.onNext(Boolean.TRUE);
                    emitter.onComplete();
                } catch (Exception e) {
                    emitter.tryOnError(e);
                }
            })
            .subscribeOn(Schedulers.io())
            .observeOn(AndroidSchedulers.mainThread())
            .subscribe(new SimpleObserver<Boolean>() {
                @Override
                public void onNext(Boolean success) {
                    ToastUtils.showShort("删除成功");
                    getData();
                }

                @Override
                public void onError(Throwable e) {
                    super.onError(e);
                    ToastUtils.showShort("删除失败");
                }
            });
}
```

# 项目组件 Java 调用示例

## Toast、确认弹窗和工具类

```java
ToastUtils.showShort("操作成功");

int padding = SizeUtils.dp2px(12);

String text = VTBTimeUtils.formatDateTime(System.currentTimeMillis());
```

```java
DialogUtil.showConfirmRreceiptDialog(
        this,
        "标题",
        "确认要执行此操作吗？",
        new ConfirmDialog.OnDialogClickListener() {
            @Override
            public void confirm() {
                deleteData(entity);
            }

            @Override
            public void cancel() {
            }
        });
```

需要更多配置时：

```java
new ConfirmDialog.Builder(this)
        .setTitleName("标题")
        .setMessage("确认要删除吗？")
        .setConfirmBtn("确定")
        .setCancelBtn("取消")
        .setOnDialogClickListener(new ConfirmDialog.OnDialogClickListener() {
            @Override
            public void confirm() {
                deleteData(entity);
            }

            @Override
            public void cancel() {
            }
        })
        .create(true)
        .show();
```

## 图片加载

圆角控件和资源配置读取 [android-xml.md](android-xml.md)。Java 只负责正常加载图片，不使用 `RoundedCorners` 模拟项目圆角方案：

```java
Glide.with(context)
        .load(imageUrl)
        .into(binding.ivCover);
```

## RadioButton 与 RadioGroup 监听

```java
binding.rbOption1.setOnCheckedChangeListener((buttonView, isChecked) -> {
    if (isChecked) {
        currentOption = OPTION_1;
        updateUi();
    }
});
```

```java
binding.rgOption.setOnCheckedChangeListener((group, checkedId) -> {
    if (checkedId == R.id.rb_option_1) {
        currentOption = OPTION_1;
    } else if (checkedId == R.id.rb_option_2) {
        currentOption = OPTION_2;
    }
    updateUi();
});
```

# Presenter 网络请求 Java 示例

```java
public class XxxListActivity
        extends BaseActivity<ActivityXxxListBinding, MainContract.Presenter>
        implements MainContract.View {
    private XxxAdapter adapter;
    private String dataUrl;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setDataBindingLayout(R.layout.activity_xxx_list);
    }

    @Override
    public void initView() {
        binding.include.setTitleStr("标题");
        dataUrl = getIntent().getStringExtra("dataUrl");

        adapter = new XxxAdapter(this, null, R.layout.item_xxx);
        binding.rv.setLayoutManager(new LinearLayoutManager(this));
        binding.rv.setAdapter(adapter);

        presenter = new MainPresenter(this);
        presenter.queryJson(dataUrl);
    }

    @Override
    public void bindEvent() {
        binding.setOnClickListener(this::onClickCallback);
    }

    @Override
    public void onClickCallback(View view) {
        if (view.getId() == R.id.iv_title_back) {
            finish();
        }
    }

    @Override
    public void queryJsonSuccess(String requestUrl, String jsonStr) {
        List<XxxEntity> list = new Gson().fromJson(
                jsonStr,
                new TypeToken<List<XxxEntity>>() {
                }.getType());
        adapter.addAllAndClear(list);
    }

    @Override
    public void hideLoading() {
        hideLoadingDialog();
    }
}
```

# 页面跳转 Java 示例

无参数跳转：

```java
skipAct(XxxActivity.class);
```

携带参数：

```java
Intent intent = new Intent(this, XxxActivity.class);
intent.putExtra("data", entity);
startActivity(intent);
```

多入口页面可提供：

```java
public static void start(Context context, XxxEntity entity) {
    Intent intent = new Intent(context, XxxActivity.class);
    intent.putExtra("data", entity);
    context.startActivity(intent);
}
```

# 自定义 Dialog Java 示例

Dialog 的公共职责读取 [android-project.md](android-project.md)，布局读取 [android-xml.md](android-xml.md)。

```java
public class XxxSelectDialog extends Dialog {
    private final DialogInterface.OnClickListener listener;
    private DialogXxxSelectBinding binding;

    public XxxSelectDialog(
            Context context,
            DialogInterface.OnClickListener listener) {
        super(context);
        this.listener = listener;
    }

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        Window dialogWindow = getWindow();
        if (dialogWindow == null) {
            return;
        }
        dialogWindow.setSoftInputMode(
                WindowManager.LayoutParams.SOFT_INPUT_ADJUST_RESIZE
                        | WindowManager.LayoutParams.SOFT_INPUT_STATE_HIDDEN);
        dialogWindow.setGravity(Gravity.CENTER);
        dialogWindow.getDecorView().setPadding(0, 0, 0, 0);
        dialogWindow.setBackgroundDrawable(new ColorDrawable(Color.TRANSPARENT));
        dialogWindow.requestFeature(Window.FEATURE_NO_TITLE);

        WindowManager.LayoutParams params = dialogWindow.getAttributes();
        params.width = ScreenUtils.getScreenWidth() - SizeUtils.dp2px(60f);
        dialogWindow.setAttributes(params);

        binding = DataBindingUtil.inflate(
                LayoutInflater.from(getContext()),
                R.layout.dialog_xxx_select,
                null,
                false);
        setContentView(binding.getRoot());

        initView();
        binding.setOnClickListener(this::onItemViewClick);
    }

    private void initView() {
        // 初始化当前 Dialog 需要的数据
    }

    private void onItemViewClick(View view) {
        switch (view.getId()) {
            case R.id.tv_option_1:
                if (listener != null) {
                    listener.onClick(this, 0);
                }
                break;
            case R.id.tv_option_2:
                if (listener != null) {
                    listener.onClick(this, 1);
                }
                break;
            default:
                break;
        }

        dismiss();
    }
}
```

显示 Dialog：

```java
new XxxSelectDialog(this, (dialog, which) -> {
    if (which == 0) {
        selectFirstItem();
    } else if (which == 1) {
        selectSecondItem();
    }
}).show();
```

# 表单处理 Java 示例

```java
private void saveData() {
    String fieldValue = binding.etField.getText().toString().trim();
    if (TextUtils.isEmpty(fieldValue)) {
        ToastUtils.showShort("请输入必填字段");
        return;
    }

    float numberValue;
    try {
        numberValue = Float.parseFloat(fieldValue);
    } catch (NumberFormatException e) {
        ToastUtils.showShort("请输入有效的数值");
        return;
    }

    if (numberValue <= 0f) {
        ToastUtils.showShort("数值必须大于0");
        return;
    }

    if (CURRENT_TYPE.equals(someType) && TextUtils.isEmpty(selectedItem)) {
        ToastUtils.showShort("请选择必选项");
        return;
    }

    saveToDatabase();
}
```

- 字符串必填验证使用 `TextUtils.isEmpty()` 或当前模块统一的空值工具，同一方法不混用多套写法。
- 数字解析只捕获 `NumberFormatException`，不使用大范围 `try/catch` 包裹整个保存方法。
