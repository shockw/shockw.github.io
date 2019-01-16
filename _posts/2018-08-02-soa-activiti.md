---
layout: post
title:  "使用jeesite快速实现业务流程开发"
categories: SOA
tags:  oa jeesite activiti 公司借款流程  
---

* content
{:toc}

企业内部协作管理的常见方式便是业务梳理并固化成流程，通过系统来实现工作流程的统一管理。笔者所在的运营商行业即有很多很多基于流程平台的业务系统，比如网络施工的统一管理、故障处理的流程管理。本文主要使用jeesit1.2.7版本进行的试验，可以在github和码云上找到此版本。

## 业务模型
```
CREATE TABLE `oa_loan` (
  `id` varchar(64) COLLATE utf8_bin NOT NULL COMMENT '编号',
  `proc_ins_id` varchar(64) COLLATE utf8_bin DEFAULT NULL COMMENT '流程实例编号',
  `user_id` varchar(64) COLLATE utf8_bin DEFAULT NULL COMMENT '用户',
  `office_id` varchar(64) COLLATE utf8_bin DEFAULT NULL COMMENT '归属部门',
  `summary` varchar(64) COLLATE utf8_bin DEFAULT NULL COMMENT '借款事由',
  `fee` int(11) DEFAULT NULL COMMENT '借款金额',
  `reason` varchar(255) COLLATE utf8_bin DEFAULT NULL COMMENT '借款原因',
  `actbank` varchar(255) COLLATE utf8_bin DEFAULT NULL COMMENT '开户行',
  `actno` varchar(255) COLLATE utf8_bin DEFAULT NULL COMMENT '账号',
  `actname` varchar(255) COLLATE utf8_bin DEFAULT NULL COMMENT '账号名',
  `financial_text` varchar(255) COLLATE utf8_bin DEFAULT NULL COMMENT '财务预审',
  `lead_text` varchar(255) COLLATE utf8_bin DEFAULT NULL COMMENT '部门领导意见',
  `main_lead_text` varchar(255) COLLATE utf8_bin DEFAULT NULL COMMENT '总经理意见',
  `teller` varchar(255) COLLATE utf8_bin DEFAULT NULL COMMENT '出纳支付',
  `create_by` varchar(64) COLLATE utf8_bin NOT NULL COMMENT '创建者',
  `create_date` datetime NOT NULL COMMENT '创建时间',
  `update_by` varchar(64) COLLATE utf8_bin NOT NULL COMMENT '更新者',
  `update_date` datetime NOT NULL COMMENT '更新时间',
  `remarks` varchar(255) COLLATE utf8_bin DEFAULT NULL COMMENT '备注信息',
  `del_flag` char(1) COLLATE utf8_bin NOT NULL DEFAULT '0' COMMENT '删除标记',
  PRIMARY KEY (`id`),
  KEY `OA_TEST_AUDIT_del_flag` (`del_flag`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='借款流程测试表';
```

## 表单开发
使用Jeesite的代码生成工具，生成业务单的增删查改界面。参考jeesite工程doc目录里的代码生成器应用。

## 流程设计
* 流程：财务审批-》部门领导审批-》总经理审批-》出纳支付
* 角色：普通用户角色 shock  
       财务初审人员 sd_zhb  
       部门经理角色 dept_leader 对应本部门中的c权限，jeesite中叫本部门管理员  
       总经理角色 shock  
       出纳员 sd_zhb
* 流程设计要点：通过activiti流程设计器设计，参考jeesiste工程doc目录里的工作流的应用实例。重点有关联表单、每个环节的主键，每个环节继续执行下去的条件，设置成条件{pass==1}。每个环节的执行人配置，可以直接填写登陆名的，也可以设置角户组。本例中的部门领导采用的是传递变量的方式，在后台通过申请人的部门查找本部门中有本部门管理员角色的用户，将此用户设置成dept_leader，则流程流转到部门经理那边。其他流程处理人员通过设置登陆名或角色来实现流程流转。
* 代码修改：需要模仿工程的用户调薪模块对之前生成的表单代码进行修改，重点是增加流程审核的功能，并需要修改业务单表的ibatis配置文件。下面是通过部门名来查找部门经理名并在流程中设置部门经理变量的过程。   
```
// 审核环节
        if ("audit".equals(taskDefKey)) {
            oaLoan.setFinancialText(oaLoan.getAct().getComment());
            if("yes".equals(oaLoan.getAct().getFlag())){
                Office office = oaLoan.getOffice();
                List<User> userList = systemService.findUser(new User(new Role("5")));
                List<User> userList1 = systemService.findUserByOfficeId(office.getId());
                userList.retainAll(userList1);
                vars.put("dept_leader", userList.get(0).getLoginName());
            }
            dao.updateFinancialText(oaLoan);
        } else if ("audit2".equals(taskDefKey)) {
```

## 测试验证
功能做完后需要多找几个账号来测试验证，在测试过程中也发现了几个activit的表少了字段，也一并修改完成。具体代码参见https://github.com/shockw/jeemanage
