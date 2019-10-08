---
tags : [spring aop, spring, java, aspect oriented programming]
---
- Spring by default uses the JDK Dynamic proxy. To use the CGLIB proxy we need to set Proxy Target Class flag as true `factory.setProxyTargetClass(true); `
- JDK Dynamic proxies can only proxy interfaces
- CGLIB proxies are actual subclasses of the proxied class

``` java
@RunWith(MockitoJUnitRunner.class)
public class AspectTestClass {

  private AspectTarget proxy;

  @Mock
  private AspectTarget aspectTarget;

  @InjectMocks
  private Aspect aspect = new Aspect();

  @Before
  public void setUp() {
    AspectTarget target = aspectTarget;
    AspectJProxyFactory factory = new AspectJProxyFactory(target);
    factory.addAspect(aspect);
    // this is set to enable CGLIB proxy in Spring
    factory.setProxyTargetClass(true);
    proxy = factory.getProxy();		
  }

  @Test
  public void testProxy() {
    //setup mock calls

    //call proxy methods		
    proxy.<<call proxy method>>();

    //verify methods of mocks injected in proxy have been called
  }
}
```
### References
- https://stackoverflow.com/questions/16047829/proxy-cannot-be-cast-to-class
- http://insufficientinformation.blogspot.com/2007/12/spring-dynamic-proxies-vs-cglib-proxies.html?_sm_au_=iVV72QFBVrdBRnQ5
