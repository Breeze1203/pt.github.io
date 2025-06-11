---
title: springmvc流程debug
date: 2025-01-03 09:10:56
categories:
 - springboot
 - java
tags: 
 - springboot
 - java
---

#### spring  mvc解读
上篇自己手写了了一下mvc的简易版，这次解读一下springmvc官方源码，先大概根据图片来了解一下流程
![springmvc.png](/upload/springmvc.png)
debug准备：实现debug的具体流程，创建springboot项目，添加web依赖，编写controller类，完成以上流程找到DispatcherServlet类（org.springframework.web.servlet包下），该类继承FrameworkServlet（抽象类），FrameworkServlet继承HttpServletBean，HttpServletBean继承HttpServlet；根据servlet的实现，肯定先执行init，找到该方法，init方法被重写了
![截屏2025-01-03 09.26.01.png](/upload/截屏2025-01-03%2009.26.01.png)

可以看到调用了父类的init，父亲类是GenericServlet，GenericServlet和HttpServlet属于不属于springmvc包下先不讨论，但这个初始化过程还是要知道什么时候开始的；
回到DispatcherServlet，里面重要的组件，与上面对应上了
```
    private MultipartResolver multipartResolver;
    @Nullable
    private LocaleResolver localeResolver;
    /** @deprecated */
    @Deprecated
    @Nullable
    private ThemeResolver themeResolver;
    @Nullable
    private List<HandlerMapping> handlerMappings;
    @Nullable
    private List<HandlerAdapter> handlerAdapters;
    @Nullable
    private List<HandlerExceptionResolver> handlerExceptionResolvers;
    @Nullable
    private RequestToViewNameTranslator viewNameTranslator;
    @Nullable
    private FlashMapManager flashMapManager;
    @Nullable
    private List<ViewResolver> viewResolvers;
```
先来看两个构造方法，第一个构造方法打上断点
```
 public DispatcherServlet() {
        this.setDispatchOptionsRequest(true);
    }

    public DispatcherServlet(WebApplicationContext webApplicationContext) {
        super(webApplicationContext);
        this.setDispatchOptionsRequest(true);
    }
```
往下走，可以看到执行到，![截屏2025-01-03 09.48.38.png](/upload/截屏2025-01-03%2009.48.38.png)
这个自动配置，这里返回的就是我们今天要讨论的DispatcherServlet；写一个发送post请求的servlet，回到HttpServlet类，给doPost打上断点，postman发送请求，会发生什么，可以看到请求的一瞬间，进入debug，执行下一步就进入FrameworkServlet的doPost，执行processRequest方法
![截屏2025-01-03 09.58.13.png](/upload/截屏2025-01-03%2009.58.13.png)

执行完毕后，走到DispatcherServlet的doDispatch方法，这里就是sprmvc执行的具体流程了。
```
 protected void doDispatch(HttpServletRequest request, HttpServletResponse response) throws Exception {
        HttpServletRequest processedRequest = request;
        HandlerExecutionChain mappedHandler = null;
        boolean multipartRequestParsed = false;
        // 提供组件异步处理 HTTP 请求
        WebAsyncManager asyncManager = WebAsyncUtils.getAsyncManager(request);

        try {
            try {
                ModelAndView mv = null;
                Exception dispatchException = null;

                try {
                    processedRequest = this.checkMultipart(request);
                    multipartRequestParsed = processedRequest != request;
                    mappedHandler = this.getHandler(processedRequest);
                    if (mappedHandler == null) {
                        this.noHandlerFound(processedRequest, response);
                        return;
                    }

                    HandlerAdapter ha = this.getHandlerAdapter(mappedHandler.getHandler());
                    String method = request.getMethod();
                    boolean isGet = HttpMethod.GET.matches(method);
                    if (isGet || HttpMethod.HEAD.matches(method)) {
                        long lastModified = ha.getLastModified(request, mappedHandler.getHandler());
                        if ((new ServletWebRequest(request, response)).checkNotModified(lastModified) && isGet) {
                            return;
                        }
                    }

                    if (!mappedHandler.applyPreHandle(processedRequest, response)) {
                        return;
                    }

                    mv = ha.handle(processedRequest, response, mappedHandler.getHandler());
                    if (asyncManager.isConcurrentHandlingStarted()) {
                        return;
                    }

                    this.applyDefaultViewName(processedRequest, mv);
                    mappedHandler.applyPostHandle(processedRequest, response, mv);
                } catch (Exception var20) {
                    dispatchException = var20;
                } catch (Throwable var21) {
                    dispatchException = new ServletException("Handler dispatch failed: " + var21, var21);
                }

                this.processDispatchResult(processedRequest, response, mappedHandler, mv, (Exception)dispatchException);
            } catch (Exception var22) {
                triggerAfterCompletion(processedRequest, response, mappedHandler, var22);
            } catch (Throwable var23) {
                triggerAfterCompletion(processedRequest, response, mappedHandler, new ServletException("Handler processing failed: " + var23, var23));
            }

        } finally {
            if (asyncManager.isConcurrentHandlingStarted()) {
                if (mappedHandler != null) {
                    mappedHandler.applyAfterConcurrentHandlingStarted(processedRequest, response);
                }

                asyncManager.setMultipartRequestParsed(multipartRequestParsed);
            } else if (multipartRequestParsed || asyncManager.isMultipartRequestParsed()) {
                this.cleanupMultipart(processedRequest);
            }

        }
    }
```
根据上图
1.第一步请求进入DispatcherServlet
2.获取Handler，返回HandlerExecutionChain，包括
![截屏2025-01-03 10.12.44.png](/upload/截屏2025-01-03%2010.12.44.png)

```
 mappedHandler = this.getHandler(processedRequest);

    @Nullable
    protected HandlerExecutionChain getHandler(HttpServletRequest request) throws Exception {
        if (this.handlerMappings != null) {
            Iterator var2 = this.handlerMappings.iterator();

            while(var2.hasNext()) {
                HandlerMapping mapping = (HandlerMapping)var2.next();
                HandlerExecutionChain handler = mapping.getHandler(request);
                if (handler != null) {
                    return handler;
                }
            }
        }
```
3 .根据返回来的Handler来寻找HandlerAdapter，继续走
```
HandlerAdapter ha = this.getHandlerAdapter(mappedHandler.getHandler());
```
 4 . HandlerAdapter处理解析到的Handler
```
  mv = ha.handle(processedRequest, response, mappedHandler.getHandler());
```
5.6.看作一步，handle方法内部，HandlerAdapter是一个接口
![截屏2025-01-03 10.24.26.png](/upload/截屏2025-01-03%2010.24.26.png)
7 返回modelview（这里的属性名为mv）
```
 // 用于给模型和视图（ModelAndView）对象mv应用默认的视图名称
 this.applyDefaultViewName(processedRequest, mv);
 // 用于执行所有后置拦截器（Interceptor）的postHandle方法
 mappedHandler.applyPostHandle(processedRequest, response, mv);
```
8 9  10 可以看成一步
```
this.processDispatchResult(processedRequest, response, mappedHandler, mv, (Exception)dispatchException);


private void processDispatchResult(HttpServletRequest request, HttpServletResponse response, @Nullable HandlerExecutionChain mappedHandler, @Nullable ModelAndView mv, @Nullable Exception exception) throws Exception {
        boolean errorView = false;
        if (exception != null) {
            if (exception instanceof ModelAndViewDefiningException) {
                ModelAndViewDefiningException mavDefiningException = (ModelAndViewDefiningException)exception;
                this.logger.debug("ModelAndViewDefiningException encountered", exception);
                mv = mavDefiningException.getModelAndView();
            } else {
                Object handler = mappedHandler != null ? mappedHandler.getHandler() : null;
                mv = this.processHandlerException(request, response, handler, exception);
                errorView = mv != null;
            }
        }

        if (mv != null && !mv.wasCleared()) {
            this.render(mv, request, response);
            if (errorView) {
                WebUtils.clearErrorRequestAttributes(request);
            }
        } else if (this.logger.isTraceEnabled()) {
            this.logger.trace("No view rendering, null ModelAndView returned.");
        }

        if (!WebAsyncUtils.getAsyncManager(request).isConcurrentHandlingStarted()) {
            if (mappedHandler != null) {
                mappedHandler.triggerAfterCompletion(request, response, (Exception)null);
            }

        }
    }

    protected LocaleContext buildLocaleContext(final HttpServletRequest request) {
        LocaleResolver lr = this.localeResolver;
        if (lr instanceof LocaleContextResolver localeContextResolver) {
            return localeContextResolver.resolveLocaleContext(request);
        } else {
            return () -> {
                return lr != null ? lr.resolveLocale(request) : request.getLocale();
            };
        }
    }         
```
可以看到这里对视图有好几种处理方式（异常，无视图，有视图），现在我们都是前后端分离，返回json，这种属于无视图的处理，有视图的话也是利用response流处理液乳到视图上，没有是直接放回response流；
![截屏2025-01-03 10.52.52.png](/upload/截屏2025-01-03%2010.52.52.png)
现在都是springboot，springboot自动配置处理了DispatcherServlet，如下
![截屏2025-01-03 09.48.38.png](/upload/截屏2025-01-03%2009.48.38.png)
