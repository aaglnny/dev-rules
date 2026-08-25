# Java Room Entity、Dao 与 Database 完整示例

`dao` 和 `entitys` 包必须使用 Java。Dao 只返回普通类型，线程切换由调用层负责。

## Entity

```java
package com.xxx.project_name.entitys;

import androidx.room.Entity;
import androidx.room.PrimaryKey;

import java.io.Serializable;

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

## Dao

```java
package com.xxx.project_name.dao;

import androidx.room.Dao;
import androidx.room.Delete;
import androidx.room.Insert;
import androidx.room.OnConflictStrategy;
import androidx.room.Query;
import androidx.room.Update;

import com.xxx.project_name.entitys.XxxEntity;

import java.util.List;

@Dao
public interface XxxDao {

    @Insert(onConflict = OnConflictStrategy.REPLACE)
    void insert(XxxEntity... beans);

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
package com.xxx.project_name.dao;

import android.content.Context;

import androidx.room.Database;
import androidx.room.Room;
import androidx.room.RoomDatabase;

import com.xxx.project_name.entitys.XxxEntity;

@Database(entities = {XxxEntity.class}, version = 1, exportSchema = false)
public abstract class DatabaseManager extends RoomDatabase {
    public static final String DB_NAME = "data.db";
    private static volatile DatabaseManager instance;

    public static DatabaseManager getInstance(Context context) {
        if (instance == null) {
            synchronized (DatabaseManager.class) {
                if (instance == null) {
                    instance = Room.databaseBuilder(
                                    context.getApplicationContext(),
                                    DatabaseManager.class,
                                    DB_NAME)
                            .allowMainThreadQueries()
                            .build();
                }
            }
        }
        return instance;
    }

    public abstract XxxDao getXxxDao();
}
```

## 实施检查

- [ ] Entity 是否无参数、实现 `Serializable`、使用 `@PrimaryKey(autoGenerate = true)`。
- [ ] SQL 表名是否与 Entity 类名一致。
- [ ] Dao 是否没有返回 `Observable`、`Single` 或 `Completable`。
- [ ] Room 注解处理器是否使用项目统一的 `annotationProcessor` 配置。
