---
title: 工作流引擎---RE
thumbnail: 'http://oayt7zau6.bkt.clouddn.com/basic.jpg'
date: 2017-05-27 11:17:39
updated: 2017-05-27 11:17:39
layout:
comments:
categories: 流程引擎
tags: 流程引擎
permalink:
---
### 部署一个流程
我们部署流程引擎的时候可以选择bpmn和zip压缩后的方式:
#### 方式一:bpmn文件的方式部署
```java
/**
 * step-one:部署流程实例
 * 
 * @return
 */
public static Deployment deploymentDefination() {
  DeploymentBuilder deploymentBuilder = processEngine.getRepositoryService().createDeployment();
  deploymentBuilder.addClasspathResource("coupon.bpmn");
  Deployment deploy = deploymentBuilder.deploy();
  return deploy;
}
```
#### 方式二:压缩文件的格式
controller层上传压缩文件
```java
/**
 * 部署一个流程
 *
 * @param bpmFile
 * @return
 */
@RequestMapping(value = "/deployments", method = RequestMethod.POST)
public Map<String, Object> createDeployment(MultipartFile bpmFile) {
  return deploymentService.createDeployment(bpmFile);
}
```
service层部署流程:<font color='red'>createDeployment方法</font>
```java
/**
 * 部署一个流程bpm文件
 *
 * @param bpmFile
 */
public Map<String, Object> createDeployment(MultipartFile bpmFile) {
    try {
        Map<String, Object> result = new HashMap<>();
        DeploymentBuilder deploymentBuilder = repositoryService.createDeployment();
        String fileName = bpmFile.getOriginalFilename();
        String type = bpmFile.getContentType();
    
        if (!StringUtils.isEmpty(fileName)) {
            if (type.equals("application/zip")) {
                ZipInputStream inputStream = new ZipInputStream(bpmFile.getInputStream());
                deploymentBuilder.addZipInputStream(inputStream);
            } else {
                throw new FlowableIllegalArgumentException("流程bmp文件格式暂不支持");
            }
    
            deploymentBuilder.name(fileName).deploy();
        }
    
        result.put("code", 0);
        result.put("message", "部署成功");
        return result;
    } catch (Exception e) {
        throw new CjjClientException(ErrorCodes.ERR_DEPLOYMENT_FAILED, "流程部署失败，bmpFie: " + bpmFile.getOriginalFilename(), e);
    }
}
```
### 查询部署的流程
controller层
```java
/**
 * 查看所有已部署流程
 *
 * @return
 */
@RequestMapping(value = "/deplyments", method = RequestMethod.GET)
public Map<String, Object> queryDeployments() {
    return deploymentService.queryDeployments();
}
```
service层
```java
public Map<String, Object> queryDeployments() {
    Map<String, Object> result = new HashMap<>();
    DeploymentQuery deploymentQuery = repositoryService.createDeploymentQuery();

    List<Deployment> deploymentList = deploymentQuery.list();
    List<DeploymentVo> deploymentVoList = new ArrayList<>();
    if (deploymentList != null && deploymentList.size() > 0) {
        for (Deployment deployment : deploymentList) {
            DeploymentVo deploymentVo = new DeploymentVo();
            deploymentVo.setId(deployment.getId());
            deploymentVo.setName(deployment.getName());
            deploymentVo.setCategory(deployment.getCategory());
            deploymentVo.setKey(deployment.getKey());
            deploymentVo.setDeploymentTime(deployment.getDeploymentTime());

            deploymentVoList.add(deploymentVo);
        }

        result.put("data", deploymentVoList);
    }
    result.put("code", 0);
    return result;
}
```
