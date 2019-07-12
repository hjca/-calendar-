# -calendar-
自定义根据vue编写的日历组件

组件中接收的参数如下：
startDate：开始日期
endDate：结束日期
如果开始日期没有值，就显示当前日期，则结束日期为开始日期交50年


父组件接受的参数：
errorMsg：报错信息
getDateVal：得到选择的日期值

案列：
<calendar-comp
    :start-date="beginDate"
    :end-date="endDate"
    @errorMag="showErrorMsg"
    @getDateVal="getDate"/>
