---
title: spring-cloud-feign在使用中遇到的一些问题
date: 2017-09-24 21:53:46
categories: spring-cloud
tags:
---

## 1. feign的接口继承特性

### 1.1 暴露的接口
    package service.sys.common.api;
    
    import com.ymu.spcselling.infrastructure.constants.SpcsConstants;
    import com.ymu.spcselling.infrastructure.idgenerator.ID;
    import org.springframework.validation.annotation.Validated;
    import org.springframework.web.bind.annotation.*;
    import service.sys.common.vo.req.VIdGenReq;
    
    /**
     * 分布式id生成服务。
     */
    @RequestMapping(SpcsConstants.API_VERSION + "/id")
    public interface IdGenerateApi {
    
        /**
         * 生成分布式id
         * @param vIdGenReq 请求对象。body体
         * @return 生成的系统全局唯一id
         *
         * @api {post} /v1/id/gen 生成分布式id
         * @apiVersion 1.0.0
         * @apiName genId
         * @apiGroup ID
         * @apiPermission admin
         *
         * @apiDescription 通过数据中心id，机器id生成long型唯一id
         *
         * @apiParam {long} dataCenterId 数据中心id,0-31。
         * @apiParam {long} workerId 机器id，0-31。
         *
         * @apiParamExample {json} Request-Example:
         *     Request Headers
         *         Content-Type:application/json
         *     body:
         *     {
         *       "dataCenterId": 0,
         *       "workerId:" 0
         *     }
         *
         * @apiExample 请求例子:
         * curl -i http://localhost/user/4711
         *
         * @apiSuccess {long}   id      生成的id
         *
         * @apiError NoAccessRight 认证不通过
         * @apiError UserNotFound   The <code>id</code> of the User was not found.
         *
         * @apiErrorExample 响应例子:
         *     HTTP/1.1 401 Not Authenticated
         *     {
         *       "error": "NoAccessRight"
         *     }
         *
         * @apiSampleRequest url
         *
         */
        @PostMapping("/gen")
        long genId(@RequestBody @Validated VIdGenReq vIdGenReq);
    
        /**
         *
         * 解析分布式id
         * @param id
         * @return
         *
         * @api {post} /v1/id/expId  解析分布式id
         * @apiVersion 1.0.0
         * @apiName expId
         * @apiGroup ID
         * @apiPermission admin
         *
         * @apiDescription 把id解析成ID对象
         *
         * @apiParam {long} id 接口生成的id，必传。
         *
         * @apiExample 请求例子:
         *  http://localhost/v1/id/expId?id=352608540609069079
         *
         * @apiSuccess {long}   timeStamp     时间戳。41位的时间序列
         * @apiSuccess {long}   dataCenterId     数据中心id
         * @apiSuccess {long}   workerId     节点机器id
         * @apiSuccess {long}   sequence     序列号
         *
         * @apiError NoAccessRight 认证不通过
         *  //@apiError UserNotFound   The <code>id</code> of the User was not found.
         *
         * @apiErrorExample 响应例子:
         *     HTTP/1.1 401 Not Authenticated
         *     {
         *       "error": "NoAccessRight"
         *     }
         *
         * @apiSampleRequest http://localhost/v1/id/expId
         *
         */
        @GetMapping("/expId")
        ID expId(@RequestParam(value = "id") long id);
    }

### 1.2 接口的实现
    package service.sys.common.controller;
    
    import com.ymu.spcselling.infrastructure.base.AbstractBaseController;
    import com.ymu.spcselling.infrastructure.idgenerator.ID;
    import org.apache.logging.log4j.LogManager;
    import org.apache.logging.log4j.Logger;
    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.cloud.context.config.annotation.RefreshScope;
    import org.springframework.web.bind.WebDataBinder;
    import org.springframework.web.bind.annotation.RequestBody;
    import org.springframework.web.bind.annotation.RestController;
    import service.sys.common.api.IdGenerateApi;
    import service.sys.common.service.local.IdService;
    import service.sys.common.vo.req.VIdGenReq;
    import service.sys.common.vo.req.VIdGenReqValidator;
    
    @RefreshScope
    @RestController
    public class IdGenerateController extends AbstractBaseController implements IdGenerateApi {
    
        private static final Logger LOGGER = LogManager.getLogger(SendEmailController.class);
    
        @Override
        protected void initBinder(WebDataBinder binder) {
            binder.addValidators(new VIdGenReqValidator());
        }
    
    
        @Autowired
        private IdService idService;
    
        @Override
        public long genId(@RequestBody VIdGenReq vIdGenReq) {
            long id = idService.genId(vIdGenReq.getDataCenterId(), vIdGenReq.getWorkerId());
            LOGGER.debug("genId:" + id);
            return id;
        }
    
        @Override
        public ID expId(long id) {
            ID ID = idService.expId(id);
            LOGGER.debug("ID=", ID.toString());
            return ID;
        }
    
    
    }
    
注意：在gen()接口方法中，虽然加了mvn的参数注解@RequestBody @Validated，但是在其实现中也要加上，否则这些注解功能将失效。
类似的，还有一些其他的注解也要加上。
> 常见的在实现中要加上的注解有：
> - @RequestBody
> - @Validated
> - @RequestHeader 
> - @RequestParam    
