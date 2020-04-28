## spring常用接口

### 1.InitializingBean接口

- 作用：实现InitializingBean接口的bean,在初始化bean时都会执行afterPropertiesSet方法
afterPropertiesSe
 
- InitializingBean接口源码如下：
``` 
public interface InitializingBean {
    void afterPropertiesSet() throws Exception;
}
```
- 使用实例：策略模式+InitializingBean
- 业务场景：调用登录接口后，需要根据接口返回的不同的错误码，做出不同响应
- 实现思路：将错误码和场景实现类的映射关系，在bean初始化时通过afterPropertiesSet初试化至Map中（仿照spring初始化beandefinition），调用时通过调用接口返回的错误码get到相应的实现类，InitializingBean在此场景中起到初始化map的作用

```
/**
 * 登录场景抽象类
 *
 * @author xiantao.wu
 * @create 2019/5/1711:32
 **/
public abstract class AbstractScene implements InitializingBean {
    private static final Map<Integer, AbstractScene> SCENE_MAP = new ConcurrentHashMap<>();
    //初始化Map
    protected void register(Integer sceneType, AbstractScene scene) {
        SCENE_MAP.put(sceneType, scene);
    }


    public static AbstractScene getScene(Integer sceneType) {
        if (sceneType == null) {
            throw 业务异常
        }
        return SCENE_MAP.get(sceneType);
    }


    //调用统一入口
    public static AppLoginResponse checkLoginResponse(LoginResponse response) {
        Integer sceneType=response.getCode;
        AbstractScene scene = getScene(sceneType);
        if (scene == null) {
           throw 业务异常
        }

        return scene.check(apiResponse);
    }

    //登录check项抽象方法
    public abstract AppLoginResponse check(ApiResponse<TokenPersonLoginResult> apiResponse);

}
```
具体实现类：

```
/**
 * 场景一:账号存在风险
 *
 * @author xiantao.wu
 * @create 2019/5/1711:35
 **/
@Service
public class RiskAccountScene extends AbstractScene {
    @Autowired
    private PersonService personService;

    @Override
    public void afterPropertiesSet() throws Exception {
        register(Errors.RISK_ACCOUNT.getErrorCode(), this);
    }

    @Override
    public AppLoginResponse check(ApiResponse<TokenPersonLoginResult> apiResponse) {
        //TODO 具体实现逻辑
    }
}

/**
 * 场景二:需要验证手机
 *
 * @author xiantao.wu
 * @create 2019/5/1711:35
 **/
@Service
public class NeedVerifyMobileScene extends AbstractScene {
    @Autowired
    private PersonService personService;

    @Override
    public void afterPropertiesSet() throws Exception {
        register(Errors.NEED_VERIFY_MOBILE.getErrorCode(), this);
    }

    @Override
    public AppLoginResponse check(ApiResponse<TokenPersonLoginResult> apiResponse) {
        //TODO 具体实现逻辑
    }
}


登录调用

 /**
     * 登录
     */
    @ApiOperation(value = "登录", tags = "PASSPORT")
    @PostMapping(value = "/login")
    public ApiResponse<AppLoginResponse> login(@RequestBody AppLoginRequest appLoginRequest) {
        LoginResponse response = loginService.login(appLoginRequest);
        return ApiResponse.success(AbstractScene.checkLoginResponse(response));
    }

```



### 2.BeanPostProcessor接口

作用：BeanPostProcessor在bean初始化前后做自定义处理

场景：需要自定义swagger,使所有请求显示固定的请求头参数(自定义Docket类)，但是swagger被架构组封装到sdk中，所以在BeanPostProcessor中对Docket的bean做处理


```
/**
 * 自定义swagger
 *
 * @author xiantao.wu
 * @create 2019/3/269:51
 **/
@Component
public class GiveAppBeanPostProcessor implements BeanPostProcessor {
    //bean初始化之前的操作
    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        return bean;
    }

    //bean初始化之后的操作
    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        //swagger统一增加请求头
        if ("docket".equals(beanName)) {
            return builderHeader((Docket) bean);
        }
        return bean;
    }

    private Docket builderHeader(Docket docket) {
        List<Parameter> pars = new TreeList();
        pars.add(new ParameterBuilder().name("x-os-name").description("系统类型").defaultValue("iOS")
                .modelRef(new ModelRef("string")).parameterType("header").required(true).build());

        pars.add(new ParameterBuilder().name("x-os-version").description("系统版本").defaultValue("10.3.3")
                .modelRef(new ModelRef("string")).parameterType("header").required(true).build());

        return docket.globalOperationParameters(pars);
    }
}

```








