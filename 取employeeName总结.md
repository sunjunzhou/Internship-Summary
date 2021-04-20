#### 总结 ---- 取字段（employeeName）

##### 一、弄清几个问题

1.该字段如何获取？（一般是通过查表，得到该字段，在其对应实体类对象里放着）

2.当前返回页对象是否已经包含，只是值为null？，还是根本就没有该字段的返回？

3.最终哪个对象携带着交给前端？

##### 二、保证该页面返回对象有该字段的返回（不考虑值）

有，不处理，后面步骤取值；没有，返回对象实体类进行定义，保证get/set到

##### 三、定位取值的来源

1.其值来自于当前页面的返回对象，直接get到

2.其值来自其他对象，需要获取后赋值给当前返回页的对象

##### 四、匹配对应值

该字段已经拿到值，要保证从其他对象获取到的值赋值给该对象时要能匹配（一一对应的条件）

拿到这个字段-------拿到需要存放的字段-------赋值存放

##### 五、返回前端

把原本就携带该字段，或是经过赋值处理后已经存在该字段的对象 return 给前端。

------

#### 对于ws项目

##### 处理流程：

###### 一、弄清几个问题

1.全局搜索定位后发现该字段来源于LeaveProcess对象。

2.返回的对象LeaveHoliday中不存在字段employeeName。

3.最终IPage<LeaveHoliday>返回给前端。

###### 二、保证该页面返回对象有该字段的返回（不考虑值）

该页面的返回对象为LeaveHoliday，但该对象中没有定义employeeName属性；

在LeaveHoliday对象实体类定义employeeName字段，保证其有set/get。

###### 三、定位取值的来源

需要返回的值来源于LeaveProcess对象，该对象需要leaveProcessService中的queryProcessListByRecords(holidayIds)方法通过传入请假单ids返回的。

那么，获取ids：

```java
//拿到分页请求返回的全体对象（此时已经有了employeeName只是值为null）
IPage<LeaveHoliday> leaveHolidayIPage = leaveHolidayMapper.selectHolidayWithEmployeePage(page, wrapper);
//获取到返回的数据，以list集合形式
List<LeaveHoliday> leaveHoliday = leaveHolidayIPage.getRecords();
//stream()形式获取ids，也可遍历后加进list集合
List<String> holidayIds = leaveHoliday.stream().map(LeaveHoliday::getId).collect(Collectors.toList())
```

需要的字段在审批单里，根据ids获取审批单

```java
//得到审批单，用于从里面get到employeeName
List<LeaveProcess> leaveProcesses = leaveProcessService.queryProcessListByRecords(holidayIds);
```

###### 四、匹配对应值

一一对应条件

```java
//审批单里的recordId等于请假单中的Id
leaveProcesses.get(j).getRecordId().equals(leaveHoliday.get(i).getId())
```

拿到需要的值 赋值给 存放的字段

```java
leaveHoliday.get(i).setEmployeeName(leaveProcesses.get(j).getEmployeeName()
```

###### 几个小点：

1.stream（）与for（）

```java
List<LeaveHoliday> leaveHoliday = leaveHolidayIPage.getRecords();
List<String> holidayIds = leaveHoliday.stream().map(LeaveHoliday::getId).collect(Collectors.toList())
```

```java
List<LeaveHoliday> leaveHoliday = leaveHolidayIPage.getRecords();
List<String> holidayIds = new ArrayList<>();
for (int i = 0; i < leaveHoliday.size(); i++) {
			holidayIds.add(leaveHoliday.get(i).getId());
		}
```

2.where in

```java
WHERE id IN
<foreach item="item" index="index" collection="list" open="(" separator="," close=")"> 
        #{item} 
 </foreach>
```

```
foreach语句中， collection属性的参数类型可以使：List、数组、map集合
​     collection： 必须跟mapper.java中@Param标签指定的元素名一样
​     item： 表示在迭代过程中每一个元素的别名，可以随便起名，但是必须跟元素中的#{}里面的名称一样。
　　 index：表示在迭代过程中每次迭代到的位置(下标)
　　 open：前缀， sql语句中集合都必须用小括号()括起来
​     close：后缀
　　 separator：分隔符，表示迭代时每个元素之间以什么分隔
```

3.LambdaQueryWrapper

```
LambdaQueryWrapper<BannerItem> wrapper = new QueryWrapper<BannerItem>().lambda();
wrapper.eq(BannerItem::getBannerId, id);
List<BannerItem> bannerItems = bannerItemMapper.selectList(wrapper);
```

```
List<BannerItem> bannerItems = new LambdaQueryChainWrapper<>(bannerItemMapper)
                        .eq(BannerItem::getBannerId, id)
                        .list();
```

```
4. StringUtils.isNotEmpty()判空
```

