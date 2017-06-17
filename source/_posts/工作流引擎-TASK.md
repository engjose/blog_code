---
title: 工作流引擎---TASK
thumbnail: 'http://oayt7zau6.bkt.clouddn.com/basic.jpg'
date: 2017-05-27 15:35:59
updated: 2017-05-27 15:35:59
layout:
comments:
categories: 流程引擎
tags: 流程引擎
permalink:
---
### 条件查询获取任务
![image description](http://oayt7zau6.bkt.clouddn.com/%E6%88%91%E7%9A%84%E5%AE%A1%E6%A0%B8%E6%9D%A1%E4%BB%B6.jpg)
1. name---根据事件名称模糊查询
2. status---根据事件状态查询
3. type根据事件类型查询

Controller层实现
```java
@RequestMapping(value = "/auditTasks", method = RequestMethod.GET)
public Map<String, Object> getAuditEvents(String name, String status, String type,
                                          @RequestParam(defaultValue = "1")int pageNo, @RequestParam(defaultValue = "20") int pageSize) {
    return taskService.getAuditEvents(name, status, type, pageNo, pageSize);
}
```
查看我的待审核列表---这里分页为静态分页,有问题
```
/**
 * 待审核列表
 *
 * @param name
 * @param status
 * @param type
 */
public Map<String, Object> getAuditEvents(String name, String status, String type, int pageNo, int pageSize) {
    Map<String, Object> map = Maps.newHashMap();
    try {
        List<String> processIdLists = Lists.newArrayList();
        List<Map<String, Object>> auditList = Lists.newArrayList();
        Map<String, String> taskProcessIdMap = Maps.newLinkedHashMap();

        //根据人查询人所在的组
        String assignee = ParameterThreadLocal.getUid();
        List<Group> groupList = identityService.createGroupQuery().groupMember(assignee).list();
        List<String> groupIdList = Lists.newArrayList();
        for (Group groupIds : groupList) {
            groupIdList.add(groupIds.getId());
        }

        //记录分页总数
        TaskQuery taskQuery = taskService.createTaskQuery().taskCandidateGroupIn(groupIdList).orderByTaskCreateTime().desc();

        //分页 listPage自带的方法不满足分页,需要手动拼成分页
        List<Task> taskList = taskQuery.listPage((pageNo - 1) == 0 ? 0 : (pageNo - 1) * pageSize, pageSize);
        if (taskList != null && taskList.size() > 0) {
            for (Task task : taskList) {
                processIdLists.add(task.getProcessInstanceId());
                taskProcessIdMap.put(task.getProcessInstanceId(), task.getId());
            }
        }
        if (processIdLists != null && processIdLists.size() > 0) {
            for (String processId : processIdLists) {
                ProcessInstanceQuery processQuery = runtimeService.createProcessInstanceQuery().processInstanceId(processId);
                if (StringUtils.isNotBlank(name)) { //模糊查询,需要手动添加 %
                    processQuery.variableValueLikeIgnoreCase("name", "%" + name + "%");
                }
                // 目前全部为待审核状态,无需过滤
//                    if (StringUtils.isNotBlank(status)) {
//                        processQuery.variableValueEqualsIgnoreCase("status", status);
//                    }
                if (StringUtils.isNotBlank(type)) {
                    processQuery.variableValueEqualsIgnoreCase("type", type);
                }

                List<ProcessInstance> processInstanceList = processQuery.list();
                //封装返回数据
                for (ProcessInstance processInstance : processInstanceList) {
                    Map<String, Object> maps = runtimeService.getVariables(processInstance.getId());

                    Map<String, Object> resultMap = Maps.newLinkedHashMap();
                    for (Map.Entry entry : maps.entrySet()) {
                        resultMap.put(String.valueOf(entry.getKey()), entry.getValue());
                    }
                    resultMap.put("taskId", Integer.valueOf(taskProcessIdMap.get(processId)));
                    //目前全部为待审核
                    resultMap.put("status", ApprovalStatus.STATUS_APPROVING);
                    auditList.add(resultMap);
                }
            }
        }
        map.put("total", taskQuery.count());
        map.put("data", auditList);
    } catch (Exception e) {
        logger.warn("Get audit events failed. error = ", e);
    }
    return map;
}
```
### 获取已驳回任务
Controller实现---权限控制,只有发起人才能看到自己驳回的任务
```java
@RequestMapping(value = "/rejectTasks", method = RequestMethod.GET)
public Map<String, Object> getRejectTasks(String eventName, String eventType, String processDefinitionKey) {
    return taskService.getRejectTasksByApplyer(eventName, eventType, processDefinitionKey);
}
```
Service实现
```java
/**
 * 查询驳回任务
 *
 * @param eventName
 * @param eventType
 * @param processDefinitionKey
 */
public Map<String, Object> getRejectTasksByApplyer(String eventName, String eventType, String
        processDefinitionKey) {

    Map<String, Object> map = Maps.newHashMap();
    List<RejectTaskVo> rejectTaskVoList = Lists.newArrayList();
    String userId = ParameterThreadLocal.getUid();

    /* 获取所有发起者是登录人的流程 */
    ProcessInstanceQuery processQuery = runtimeService.createProcessInstanceQuery().processDefinitionKey(processDefinitionKey)
            .variableValueEqualsIgnoreCase("starter", userId);
    if (!StringUtils.isEmpty(eventName)) {
        processQuery.variableValueLikeIgnoreCase("name", "%" + eventName + "%");
    }
    if (!StringUtils.isEmpty(eventType)) {
        processQuery.variableValueLikeIgnoreCase("type", "%" + eventType + "%");
    }
    List<ProcessInstance> processInstanceList = processQuery.list();

    for (ProcessInstance processInstance : processInstanceList) {
        List<Comment> comments = taskService.getProcessInstanceComments(processInstance.getId());
        if (!comments.isEmpty() && "reject".equals(comments.get(0).getType())) {
            String currentTaskId = taskService.createTaskQuery().processInstanceId(processInstance.getId()).orderByTaskCreateTime().asc().listPage(0, 1).get(0).getId();
            Comment comment = comments.get(comments.size() - 1);
            Map variables = runtimeService.getVariables(processInstance.getId());
            RejectTaskVo rejectTaskVo = new RejectTaskVo();
            rejectTaskVo.setEventId(Integer.valueOf((String) variables.get("id")));
            rejectTaskVo.setEventName((String) variables.get("name"));
            rejectTaskVo.setEventType((String) variables.get("type"));
            rejectTaskVo.setCurrentTaskId(currentTaskId);
            rejectTaskVo.setRejectPerson(comment.getUserId());
            rejectTaskVo.setRejectTaskId(comment.getTaskId());
            rejectTaskVo.setRejectTime(comment.getTime());
            rejectTaskVo.setVariables(variables);
            rejectTaskVoList.add(rejectTaskVo);
        }
    }
    map.put("rejectTasks", rejectTaskVoList);
    return map;
}
```
### 获取某个任务的评论
Controller实现
```java
@RequestMapping(value = "/getComment", method = RequestMethod.GET)
public String getComment(String taskId) {
    return taskService.getCommentMessageByTaskId(taskId);
}
```
Service实现
```java
/**
 * 根据taskId获取流程评论信息
 *
 * @param taskId
 * @return
 */
public String getCommentMessageByTaskId(String taskId) {

    Comment comment = getCommentByTaskId(taskId);
    if (null == comment) {
        return "";
    } else {
        return comment.getFullMessage();
    }
}

/**
 * 根据taskId获取流程评论
 *
 * @param taskId
 * @return
 */
private Comment getCommentByTaskId(String taskId) {
    
    List<Comment> commentList = taskService.getTaskComments(taskId, "reject");
    if(commentList.isEmpty()) {
        return null;
    } else {
        return commentList.get(0);
    }
}
```
### 驳回任务事件
Controller实现
```java
/**
 * 驳回任务
 *
 * @param taskId
 * @param completeTaskForm
 */
@RequestMapping(value = "/{taskId}/reverse", method = RequestMethod.POST)
public String rejectTask(@PathVariable String taskId, @RequestBody CompleteTaskForm completeTaskForm) {
    return taskService.rejectTask(taskId, completeTaskForm);
}
```
Service实现
```java
/**
 * 驳回
 *
 * @param taskId
 * @param completeTaskForm
 */
public String rejectTask(String taskId, CompleteTaskForm completeTaskForm) {
    Map<String, Object> map = new HashMap<>();
    if (StringUtils.isNotBlank(taskId) && null != completeTaskForm) {
        if (StringUtils.isBlank(completeTaskForm.getComment())) { //驳回的时候,驳回原因为必填项
            throw new CjjClientException(ErrorCodes.ERR_REJECT_COMMENT_FAILED, ErrorMsgs.ERR_REVERSE_COMMENT_MSG);
        }
        String processInstanceId = taskService.createTaskQuery().taskId(taskId).singleResult().getProcessInstanceId();

        String actor = ParameterThreadLocal.getUid();
        //添加当前任务的完成者
        Authentication.setAuthenticatedUserId(actor);
        //添加批注信息
        taskService.addComment(taskId, processInstanceId, "reject", completeTaskForm.getComment());
        String taskDefinitionKey = taskService.createTaskQuery().taskId(taskId).singleResult().getTaskDefinitionKey();
        map.put(taskDefinitionKey + "_message", "no");

        //完成任务
        taskService.complete(taskId, map);
        return "success";
    } else {
        return "error";
    }
}
```
### 完成任务实现
```java
/**
 * 完成任务
 *
 * @param taskId
 * @param completeTaskForm
 */
public String completeTask(String taskId, CompleteTaskForm completeTaskForm) {
    Map<String, Object> map = new HashMap<>();
    if (StringUtils.isNotBlank(taskId) && null != completeTaskForm) {
        String processInstanceId = taskService.createTaskQuery().taskId(taskId).singleResult().getProcessInstanceId();
        //添加当前任务的完成者
        String actor = ParameterThreadLocal.getUid();
        Authentication.setAuthenticatedUserId(actor);
        taskService.addComment(taskId, processInstanceId, "complete", "已完成");
        String taskDefinitionKey = taskService.createTaskQuery().taskId(taskId).singleResult().getTaskDefinitionKey();
        map.put(taskDefinitionKey + "_message", "yes");
        //将可能更新后的变量放入变量列表中
        if (completeTaskForm.getVariables() != null && completeTaskForm.getVariables().size() > 0) {
            map.putAll(completeTaskForm.getVariables());
        }

        //完成任务
        taskService.complete(taskId, map);
        return "success";
    } else {
        return "error";
    }
}
```


