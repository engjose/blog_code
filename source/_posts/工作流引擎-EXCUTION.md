---
title: 工作流引擎---EXCUTION
thumbnail: 'http://oayt7zau6.bkt.clouddn.com/basic.jpg'
date: 2017-05-27 14:59:30
updated: 2017-05-27 14:59:30
layout:
comments: 流程引擎
categories: 流程引擎
tags:
permalink:
---
EXCUTION其实就是流程启动后对应的一个实例,每一个流程对应一个,在一个独立的流程中process(excution)--ID是唯一的
### 启动流程实例
当我们部署好了之后就需要来启动流程实例,当我们以key的方式启动时,默认的会启动最后一次的部署
```java
String processDefinitionKey = "helloActivitiProcess";
ProcessInstance pi = processEngine.getRuntimeService().startProcessInstanceByKey(processDefinitionKey); 
```
### 获取流程图,并高亮节点
当我们流程启动并在运行的节点中,这时候为了全局的观测流程运行的进度,就需要以流程图片的方式展现,并且当前节点需要高亮.Controller层实现:
```java
/**
 * 获取流程框图
 *
 * @param processInstanceId
 * @param response
 * @return
 */
@RequestMapping(value = "/{processInstanceId}/diagram", method = RequestMethod.GET)
public Map<String, Object> getProcessDiagram(@PathVariable String processInstanceId, HttpServletResponse response) {
    Map<String, Object> processDiagram = processInstanceService.getProcessDiagram(processInstanceId);
    if (processDiagram != null && processDiagram.size() > 0) {
        String pngName = "flowengine-"+ processInstanceId +".png";
        if (StringUtils.isNotBlank((String) processDiagram.get("pngName"))) {
            pngName = (String) processDiagram.get("pngName");
        }
        if (processDiagram.get("pngStream") != null) {
            processInstanceService.downFile((InputStream) processDiagram.get("pngStream"), pngName, response);
        }
    }
    return null;
}
```
<font color='red'>rocessInstanceService.getProcessDiagram(processInstanceId);</font>方法
```java
/**
 * 获取流程状态图形化展示
 *
 * @param processInstanceId
 * @return
 */
public Map<String, Object> getProcessDiagram(String processInstanceId) {
    if (StringUtils.isNotBlank(processInstanceId)) {
        Map<String, Object> result = new HashMap<String, Object>();
        TaskQuery query = taskService.createTaskQuery();
        Task task = query.processInstanceId(processInstanceId).singleResult();
        if (task != null) {
            //获取下载文件名称
            ProcessInstance instance = runtimeService.createProcessInstanceQuery()
                    .processInstanceId(processInstanceId).singleResult();
            ProcessDefinition definition = repositoryService.getProcessDefinition(instance.getProcessDefinitionId());
            if (definition != null) {
                result.put("pngName", definition.getDiagramResourceName());
            }

            // 获取活动节点
            BpmnModel bpmnModel = repositoryService.getBpmnModel(task.getProcessDefinitionId());
            List<String> activityIds = runtimeService.getActiveActivityIds(task.getExecutionId());
            InputStream inputStream = processEngine.getProcessEngineConfiguration()
                    .getProcessDiagramGenerator()
                    .generateDiagram(bpmnModel, "png", activityIds, new ArrayList(),
                            processEngine.getProcessEngineConfiguration().getActivityFontName(),
                            processEngine.getProcessEngineConfiguration().getLabelFontName(), null, null, 1.0);
            result.put("pngStream", inputStream);
            return result;
        }
    }
    return null;
}
```
下载图片方法:<font color='red'>processInstanceService.downFile</font>
```java
/**
 * 下载png图片
 *
 * @param pngStream
 * @param response
 */
public void downFile(InputStream pngStream, String pngName, HttpServletResponse response) {
    if (pngStream != null) {
        byte[] buffer = new byte[1024];
        BufferedInputStream bis = new BufferedInputStream(pngStream);
        OutputStream os = null;

        response.setContentType("image/png");
        response.addHeader("Content-Disposition", "attachment;fileName=" + pngName);
        try {
            os = response.getOutputStream();
            int i = bis.read(buffer);
            while (i != -1) {
                os.write(buffer, 0, i);
                i = bis.read(buffer);
            }
        } catch (IOException e) {
            logger.error("output image error", e);
        } finally {
            if (os != null) {
                try {
                    os.close();
                } catch (IOException e) {
                    logger.warn("close stream resource failed ", e);
                }
            }
            if (bis != null) {
                try {
                    bis.close();
                } catch (IOException e) {
                    logger.warn("close stream resource failed ", e);
                }
            }
        }
    }
}
```
### 根据条件查询流程实例
```java
/**
 * 查询processIntances
 *
 * @param queryRequest
 * @return
 */
public List<ProcessInstanceVo> queryProcessInstances(ProcessInstanceQueryForm queryRequest) {
    ProcessInstanceQuery query = runtimeService.createProcessInstanceQuery();

    if (queryRequest.getProcessInstanceId() != null) {
        query.processInstanceId(queryRequest.getProcessInstanceId());
    }
    if (queryRequest.getProcessDefinitionKey() != null) {
        query.processDefinitionKey(queryRequest.getProcessDefinitionKey());
    }
    if (queryRequest.getProcessDefinitionId() != null) {
        query.processDefinitionId(queryRequest.getProcessDefinitionId());
    }
    if (queryRequest.getProcessBusinessKey() != null) {
        query.processInstanceBusinessKey(queryRequest.getProcessBusinessKey());
    }
    if (queryRequest.getVariables() != null) {
        addVariables(query, queryRequest.getVariables());
    }

    List<ProcessInstanceVo> instanceList = new ArrayList<>();

    List<ProcessInstance> processInstances = query.list();
    for (ProcessInstance processInstance : processInstances) {
        ProcessInstanceVo processInstanceVo = new ProcessInstanceVo();

        processInstanceVo.setProcessInstanceId(processInstance.getProcessInstanceId());
        processInstanceVo.setProcessDefinitionId(processInstance.getProcessDefinitionId());
        processInstanceVo.setProcessDefinitionKey(processInstance.getProcessDefinitionKey());
        processInstanceVo.setProcessBusinessKey(processInstance.getBusinessKey());
        processInstanceVo.setVariables(processInstance.getProcessVariables());

        instanceList.add(processInstanceVo);
    }

    return instanceList;
}
```
