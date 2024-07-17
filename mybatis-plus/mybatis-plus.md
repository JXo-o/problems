#### 问题1：自动填充的数据在部分情况下不生效，如逻辑删除

```java
@Component
public class MybatisMetaObjectHandler implements MetaObjectHandler {
    @Override
    public void insertFill(MetaObject metaObject) {
        this.strictInsertFill(metaObject, "createTime", Date.class, new Date());
        this.strictInsertFill(metaObject, "updateTime", Date.class, new Date());
    }

    @Override
    public void updateFill(MetaObject metaObject) {
        //this.setFieldValByName("updateTime", new Date(), metaObject);
        this.strictUpdateFill(metaObject, "updateTime", Date.class, new Date());
    }
}

```

需要传入实体时候才会生效，默认实体属性为null时不更新，自动填充的原理是将数据放到实体内进行插入。

