---
title: Spring的初始化（下）
date: 2018-04-16 15:33:05
tags:
  - Spring
categories:
    - Spring
type: tags
---
### Spring的初始化（下）
上篇说道了bean的初始化，如何产生bean，现在轮到对bean中的属性进行注入的时候了，根据Spring的生命周期，还需要对Spring的扩展接口进行分析。
### 属性注入
所谓的属性注入，按照我的理解就是对产生的对象调用其setter方法给属性赋值。  
之前代码分析到doCreateBean方法，在AbstractAutowireCapableBeanFactory类中。我们可以发现一个populateBean方法。

```
 protected void populateBean(String beanName, RootBeanDefinition mbd, BeanWrapper bw) {
        Object pvs = mbd.getPropertyValues();
        if(bw == null) {
            if(!((PropertyValues)pvs).isEmpty()) {
                throw new BeanCreationException(mbd.getResourceDescription(), beanName, "Cannot apply property values to null instance");
            }
        } else {
            boolean continueWithPropertyPopulation = true;
            if(!mbd.isSynthetic() && this.hasInstantiationAwareBeanPostProcessors()) {
                Iterator hasInstAwareBpps = this.getBeanPostProcessors().iterator();

                while(hasInstAwareBpps.hasNext()) {
                    BeanPostProcessor needsDepCheck = (BeanPostProcessor)hasInstAwareBpps.next();
                    if(needsDepCheck instanceof InstantiationAwareBeanPostProcessor) {
                        InstantiationAwareBeanPostProcessor filteredPds = (InstantiationAwareBeanPostProcessor)needsDepCheck;
                        if(!filteredPds.postProcessAfterInstantiation(bw.getWrappedInstance(), beanName)) {
                            continueWithPropertyPopulation = false;
                            break;
                        }
                    }
                }
            }

            if(continueWithPropertyPopulation) {
                if(mbd.getResolvedAutowireMode() == 1 || mbd.getResolvedAutowireMode() == 2) {
                    MutablePropertyValues hasInstAwareBpps1 = new MutablePropertyValues((PropertyValues)pvs);
                    if(mbd.getResolvedAutowireMode() == 1) {
                        this.autowireByName(beanName, mbd, bw, hasInstAwareBpps1);
                    }

                    if(mbd.getResolvedAutowireMode() == 2) {
                        this.autowireByType(beanName, mbd, bw, hasInstAwareBpps1);
                    }

                    pvs = hasInstAwareBpps1;
                }

                boolean hasInstAwareBpps2 = this.hasInstantiationAwareBeanPostProcessors();
                boolean needsDepCheck1 = mbd.getDependencyCheck() != 0;
                if(hasInstAwareBpps2 || needsDepCheck1) {
                    PropertyDescriptor[] filteredPds1 = this.filterPropertyDescriptorsForDependencyCheck(bw, mbd.allowCaching);
                    if(hasInstAwareBpps2) {
                        Iterator var9 = this.getBeanPostProcessors().iterator();

                        while(var9.hasNext()) {
                            BeanPostProcessor bp = (BeanPostProcessor)var9.next();
                            if(bp instanceof InstantiationAwareBeanPostProcessor) {
                                InstantiationAwareBeanPostProcessor ibp = (InstantiationAwareBeanPostProcessor)bp;
                                pvs = ibp.postProcessPropertyValues((PropertyValues)pvs, filteredPds1, bw.getWrappedInstance(), beanName);
                                if(pvs == null) {
                                    return;
                                }
                            }
                        }
                    }

                    if(needsDepCheck1) {
                        this.checkDependencies(beanName, mbd, filteredPds1, (PropertyValues)pvs);
                    }
                }

                this.applyPropertyValues(beanName, mbd, bw, (PropertyValues)pvs);
            }
        }
    }
```
上面这段代码是拿到MutablePropertyValues变量，然后调用applyPropertyValues方法进行属性赋值。
点击进去查看applyPropertyValues方法：

```
 protected void applyPropertyValues(String beanName, BeanDefinition mbd, BeanWrapper bw, PropertyValues pvs) {
        if(pvs != null && !pvs.isEmpty()) {
            MutablePropertyValues mpvs = null;
            if(System.getSecurityManager() != null && bw instanceof BeanWrapperImpl) {
                ((BeanWrapperImpl)bw).setSecurityContext(this.getAccessControlContext());
            }

            List original;
            if(pvs instanceof MutablePropertyValues) {
                mpvs = (MutablePropertyValues)pvs;
                if(mpvs.isConverted()) {
                    try {
                        bw.setPropertyValues(mpvs);
                        return;
                    } catch (BeansException var18) {
                        throw new BeanCreationException(mbd.getResourceDescription(), beanName, "Error setting property values", var18);
                    }
                }

                original = mpvs.getPropertyValueList();
            } else {
                original = Arrays.asList(pvs.getPropertyValues());
            }

            Object converter = this.getCustomTypeConverter();
            if(converter == null) {
                converter = bw;
            }

            BeanDefinitionValueResolver valueResolver = new BeanDefinitionValueResolver(this, beanName, mbd, (TypeConverter)converter);
            ArrayList deepCopy = new ArrayList(original.size());
            boolean resolveNecessary = false;
            Iterator ex = original.iterator();

            while(true) {
                while(ex.hasNext()) {
                    PropertyValue pv = (PropertyValue)ex.next();
                    if(pv.isConverted()) {
                        deepCopy.add(pv);
                    } else {
                        String propertyName = pv.getName();
                        Object originalValue = pv.getValue();
                        Object resolvedValue = valueResolver.resolveValueIfNecessary(pv, originalValue);
                        Object convertedValue = resolvedValue;
                        boolean convertible = bw.isWritableProperty(propertyName) && !PropertyAccessorUtils.isNestedOrIndexedProperty(propertyName);
                        if(convertible) {
                            convertedValue = this.convertForProperty(resolvedValue, propertyName, bw, (TypeConverter)converter);
                        }

                        if(resolvedValue == originalValue) {
                            if(convertible) {
                                pv.setConvertedValue(convertedValue);
                            }

                            deepCopy.add(pv);
                        } else if(convertible && originalValue instanceof TypedStringValue && !((TypedStringValue)originalValue).isDynamic() && !(convertedValue instanceof Collection) && !ObjectUtils.isArray(convertedValue)) {
                            pv.setConvertedValue(convertedValue);
                            deepCopy.add(pv);
                        } else {
                            resolveNecessary = true;
                            deepCopy.add(new PropertyValue(pv, convertedValue));
                        }
                    }
                }

                if(mpvs != null && !resolveNecessary) {
                    mpvs.setConverted();
                }

                try {
                    bw.setPropertyValues(new MutablePropertyValues(deepCopy));
                    return;
                } catch (BeansException var19) {
                    throw new BeanCreationException(mbd.getResourceDescription(), beanName, "Error setting property values", var19);
                }
            }
        }
    }
```
这段代码的大致意思遍历当前的deepCopy,拿到每一个PropertyValue，通过BeanWrapper类的setPropertyValues方法进行操作。中间代码段太多了，截取了一段核心赋值代码，setValue方法。

```
        public void setValue(final Object object, Object valueToApply) throws Exception {
            final Method writeMethod = this.pd instanceof GenericTypeAwarePropertyDescriptor?((GenericTypeAwarePropertyDescriptor)this.pd).getWriteMethodForActualAccess():this.pd.getWriteMethod();
            if(!Modifier.isPublic(writeMethod.getDeclaringClass().getModifiers()) && !writeMethod.isAccessible()) {
                if(System.getSecurityManager() != null) {
                    AccessController.doPrivileged(new PrivilegedAction() {
                        public Object run() {
                            writeMethod.setAccessible(true);
                            return null;
                        }
                    });
                } else {
                    writeMethod.setAccessible(true);
                }
            }

            final Object value = valueToApply;
            if(System.getSecurityManager() != null) {
                try {
                    AccessController.doPrivileged(new PrivilegedExceptionAction() {
                        public Object run() throws Exception {
                            writeMethod.invoke(object, new Object[]{value});
                            return null;
                        }
                    }, BeanWrapperImpl.this.acc);
                } catch (PrivilegedActionException var6) {
                    throw var6.getException();
                }
            } else {
                writeMethod.invoke(BeanWrapperImpl.this.getWrappedInstance(), new Object[]{valueToApply});
            }

        }
    }
}
```
这段代码是通过反射给Bean属性赋值。
### 接口注入
属性注入完成之后，可以对扩展的接口如BeanFactoryAware，BeanNameAware进行注入。 首先来看initializeBean方法:

```
  protected Object initializeBean(final String beanName, final Object bean, RootBeanDefinition mbd) {
        if(System.getSecurityManager() != null) {
            AccessController.doPrivileged(new PrivilegedAction() {
                public Object run() {
                    AbstractAutowireCapableBeanFactory.this.invokeAwareMethods(beanName, bean);
                    return null;
                }
            }, this.getAccessControlContext());
        } else {
            this.invokeAwareMethods(beanName, bean);
        }

        Object wrappedBean = bean;
        if(mbd == null || !mbd.isSynthetic()) {
            wrappedBean = this.applyBeanPostProcessorsBeforeInitialization(bean, beanName);
        }

        try {
            this.invokeInitMethods(beanName, wrappedBean, mbd);
        } catch (Throwable var6) {
            throw new BeanCreationException(mbd != null?mbd.getResourceDescription():null, beanName, "Invocation of init method failed", var6);
        }

        if(mbd == null || !mbd.isSynthetic()) {
            wrappedBean = this.applyBeanPostProcessorsAfterInitialization(wrappedBean, beanName);
        }

        return wrappedBean;
    }
```
其中有一个invokeAwareMethods方法：

```
   private void invokeAwareMethods(String beanName, Object bean) {
        if(bean instanceof Aware) {
            if(bean instanceof BeanNameAware) {
                ((BeanNameAware)bean).setBeanName(beanName);
            }

            if(bean instanceof BeanClassLoaderAware) {
                ((BeanClassLoaderAware)bean).setBeanClassLoader(this.getBeanClassLoader());
            }

            if(bean instanceof BeanFactoryAware) {
                ((BeanFactoryAware)bean).setBeanFactory(this);
            }
        }

    }
```
bean如果实现了BeanNameAware接口。则给bean的name进行赋值。如果实现了BeanClassLoaderAware接口，则设置当前的BeanClassLoaderAware。如果当前当前实现了BeanFactoryAware接口，则设置setBeanFactory为当前的Factory.
接下来调用invokeInitMethods方法：

```
protected void invokeInitMethods(String beanName, final Object bean, RootBeanDefinition mbd) throws Throwable {
        boolean isInitializingBean = bean instanceof InitializingBean;
        if(isInitializingBean && (mbd == null || !mbd.isExternallyManagedInitMethod("afterPropertiesSet"))) {
            if(this.logger.isDebugEnabled()) {
                this.logger.debug("Invoking afterPropertiesSet() on bean with name \'" + beanName + "\'");
            }

            if(System.getSecurityManager() != null) {
                try {
                    AccessController.doPrivileged(new PrivilegedExceptionAction() {
                        public Object run() throws Exception {
                            ((InitializingBean)bean).afterPropertiesSet();
                            return null;
                        }
                    }, this.getAccessControlContext());
                } catch (PrivilegedActionException var6) {
                    throw var6.getException();
                }
            } else {
                ((InitializingBean)bean).afterPropertiesSet();
            }
        }

        if(mbd != null) {
            String initMethodName = mbd.getInitMethodName();
            if(initMethodName != null && (!isInitializingBean || !"afterPropertiesSet".equals(initMethodName)) && !mbd.isExternallyManagedInitMethod(initMethodName)) {
                this.invokeCustomInitMethod(beanName, bean, mbd);
            }
        }

    }
```
可以看到调用afterPropertiesSet()方法，接着尝试去哪init-method，假如有的话，通过反射，调用invokeCustomInitMethod方法。  
继续往下走，可以看到调用applyBeanPostProcessorsAfterInitialization方法。

```
  public Object applyBeanPostProcessorsAfterInitialization(Object existingBean, String beanName) throws BeansException {
        Object result = existingBean;
        Iterator var4 = this.getBeanPostProcessors().iterator();

        do {
            if(!var4.hasNext()) {
                return result;
            }

            BeanPostProcessor beanProcessor = (BeanPostProcessor)var4.next();
            result = beanProcessor.postProcessAfterInitialization(result, beanName);
        } while(result != null);

        return result;
    }
```
通过遍历BeanPostProcessor的列表，调用postProcessAfterInitialization方法。
### 总结
bean的初始化从配置文件读入到对象的产生,整体流程就是这些，通过具体的分析可以得知bean是严格按照生命周期执行的，我们可以看源码的时候可以一部分拆开来看，多看几次就知道代码在做什么了。





