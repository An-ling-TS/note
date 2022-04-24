当一个员工被维护了多个岗位时，由于员工id的重复会导致渲染问题

![image-20210825185038512](C:\Users\Ric\AppData\Roaming\Typora\typora-user-images\image-20210825185038512.png)

解决：

在视图中新配置一个index索引列，值为当前页当前行的索引，不同页之间可以重复

hzero中，将XXXId修改为XXXIndex

```js
<Form.Item
    label={intl.get(`${commonPromptCode}.reporter.CQ`).d('提交人')}
    {...SEARCH_FORM_ITEM_LAYOUT}
>
    {getFieldDecorator('reporterIndex', {})(
        <Lov
            code="HALM.EMPLOYEE_ORG"
            queryParams={{ tenantId }}
            onChange={(val, record) => {
                const { registerField, setFieldsValue } = this.props.form;
                registerField('reporterId');
                this.props.form.setFieldsValue({ reporterId: record.employeeId })
            }}
        />
    )}
</Form.Item>
```

另外注册xxxId并显式赋值