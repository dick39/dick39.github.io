#Spring IoC容器学习之旅
###在读《Spring技术内幕》，对第二章进行自己的理解总结，还在更新中。。。

##接口

##使用步骤
1.BeanDefinition的
##学习
首先我从最常用的FileSystemXmlApplicationContext开始看起。
```java
public class FileSystemXmlApplicationContext extends AbstractXmlApplicationContext{
    public FileSystemXmlApplicationContext(String[] configLocations, boolean refresh, ApplicationContext parent)
            throws BeansException {
        super(parent);
        setConfigLocations(configLocations);
        if (refresh) {
            refresh();
        }
    }
}
```
```sequence
FileSystemXmlApplicationContext->AbstractApplicationContext:refresh()
AbstractApplicationContext->AbstractRefreshableApplicationContext:refresh()
AbstractRefreshableApplicationContext->DefaultListableBeanFactory:createBeanFactory()
AbstractRefreshableApplicationContext->XmlBeanDefinitionReader:loadBeanDefinitions()
XmlBeanDefinitionReader->BeanDefinitionParserDelegate:parseBeanDefinitionElement()
```