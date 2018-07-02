---
title: Spring源码解析（bean的初始化过程（上））
date: 2018-04-04 14:47:27
tags:
  - Spring
categories:
    - Spring
type: tags
---

###  Bean的构造
Bean的初始化需要从AbstractBeanFactory这个类说起，首先调用doGetBean方法，这里只讨论单例的bean，来看下该方法的实现：

```
    protected <T> T doGetBean(String name, Class<T> requiredType, final Object[] args, boolean typeCheckOnly) throws BeansException {
        final String beanName = this.transformedBeanName(name);
        Object sharedInstance = this.getSingleton(beanName);
        Object bean;
        if(sharedInstance != null && args == null) {
            if(this.logger.isDebugEnabled()) {
                if(this.isSingletonCurrentlyInCreation(beanName)) {
                    this.logger.debug("Returning eagerly cached instance of singleton bean \'" + beanName + "\' that is not fully initialized yet - a consequence of a circular reference");
                } else {
                    this.logger.debug("Returning cached instance of singleton bean \'" + beanName + "\'");
                }
            }

            bean = this.getObjectForBeanInstance(sharedInstance, name, beanName, (RootBeanDefinition)null);
        } else {
            if(this.isPrototypeCurrentlyInCreation(beanName)) {
                throw new BeanCurrentlyInCreationException(beanName);
            }

            BeanFactory ex = this.getParentBeanFactory();
            if(ex != null && !this.containsBeanDefinition(beanName)) {
                String var24 = this.originalBeanName(name);
                if(args != null) {
                    return ex.getBean(var24, args);
                }

                return ex.getBean(var24, requiredType);
            }

            if(!typeCheckOnly) {
                this.markBeanAsCreated(beanName);
            }

            try {
                final RootBeanDefinition ex1 = this.getMergedLocalBeanDefinition(beanName);
                this.checkMergedBeanDefinition(ex1, beanName, args);
                String[] dependsOn = ex1.getDependsOn();
                String[] scopeName;
                if(dependsOn != null) {
                    scopeName = dependsOn;
                    int scope = dependsOn.length;

                    for(int ex2 = 0; ex2 < scope; ++ex2) {
                        String dependsOnBean = scopeName[ex2];
                        if(this.isDependent(beanName, dependsOnBean)) {
                            throw new BeanCreationException(ex1.getResourceDescription(), beanName, "Circular depends-on relationship between \'" + beanName + "\' and \'" + dependsOnBean + "\'");
                        }

                        this.registerDependentBean(dependsOnBean, beanName);
                        this.getBean(dependsOnBean);
                    }
                }

                if(ex1.isSingleton()) {
                    sharedInstance = this.getSingleton(beanName, new ObjectFactory() {
                        public Object getObject() throws BeansException {
                            try {
                                return AbstractBeanFactory.this.createBean(beanName, ex1, args);
                            } catch (BeansException var2) {
                                AbstractBeanFactory.this.destroySingleton(beanName);
                                throw var2;
                            }
                        }
                    });
                    bean = this.getObjectForBeanInstance(sharedInstance, name, beanName, ex1);
                } else if(ex1.isPrototype()) {
                    scopeName = null;

                    Object var25;
                    try {
                        this.beforePrototypeCreation(beanName);
                        var25 = this.createBean(beanName, ex1, args);
                    } finally {
                        this.afterPrototypeCreation(beanName);
                    }

                    bean = this.getObjectForBeanInstance(var25, name, beanName, ex1);
                } else {
                    String var26 = ex1.getScope();
                    Scope var27 = (Scope)this.scopes.get(var26);
                    if(var27 == null) {
                        throw new IllegalStateException("No Scope registered for scope name \'" + var26 + "\'");
                    }

                    try {
                        Object var28 = var27.get(beanName, new ObjectFactory() {
                            public Object getObject() throws BeansException {
                                AbstractBeanFactory.this.beforePrototypeCreation(beanName);

                                Object var1;
                                try {
                                    var1 = AbstractBeanFactory.this.createBean(beanName, ex1, args);
                                } finally {
                                    AbstractBeanFactory.this.afterPrototypeCreation(beanName);
                                }

                                return var1;
                            }
                        });
                        bean = this.getObjectForBeanInstance(var28, name, beanName, ex1);
                    } catch (IllegalStateException var21) {
                        throw new BeanCreationException(beanName, "Scope \'" + var26 + "\' is not active for the current thread; consider " + "defining a scoped proxy for this bean if you intend to refer to it from a singleton", var21);
                    }
                }
            } catch (BeansException var23) {
                this.cleanupAfterBeanCreationFailure(beanName);
                throw var23;
            }
        }

        if(requiredType != null && bean != null && !requiredType.isAssignableFrom(bean.getClass())) {
            try {
                return this.getTypeConverter().convertIfNecessary(bean, requiredType);
            } catch (TypeMismatchException var22) {
                if(this.logger.isDebugEnabled()) {
                    this.logger.debug("Failed to convert bean \'" + name + "\' to required type [" + ClassUtils.getQualifiedName(requiredType) + "]", var22);
                }

                throw new BeanNotOfRequiredTypeException(name, requiredType, bean.getClass());
            }
        } else {
            return bean;
        }
    }
```
可以看到单例的bean需要调用到createBean方法。来看下该方法的具体的实现，它最终是调用AbstractAutowireCapableBeanFactory的createBean方法：

```
   protected Object createBean(String beanName, RootBeanDefinition mbd, Object[] args) throws BeanCreationException {
        if(this.logger.isDebugEnabled()) {
            this.logger.debug("Creating instance of bean \'" + beanName + "\'");
        }

        RootBeanDefinition mbdToUse = mbd;
        Class resolvedClass = this.resolveBeanClass(mbd, beanName, new Class[0]);
        if(resolvedClass != null && !mbd.hasBeanClass() && mbd.getBeanClassName() != null) {
            mbdToUse = new RootBeanDefinition(mbd);
            mbdToUse.setBeanClass(resolvedClass);
        }

        try {
            mbdToUse.prepareMethodOverrides();
        } catch (BeanDefinitionValidationException var7) {
            throw new BeanDefinitionStoreException(mbdToUse.getResourceDescription(), beanName, "Validation of method overrides failed", var7);
        }

        Object beanInstance;
        try {
            beanInstance = this.resolveBeforeInstantiation(beanName, mbdToUse);
            if(beanInstance != null) {
                return beanInstance;
            }
        } catch (Throwable var8) {
            throw new BeanCreationException(mbdToUse.getResourceDescription(), beanName, "BeanPostProcessor before instantiation of bean failed", var8);
        }

        beanInstance = this.doCreateBean(beanName, mbdToUse, args);
        if(this.logger.isDebugEnabled()) {
            this.logger.debug("Finished creating instance of bean \'" + beanName + "\'");
        }

        return beanInstance;
    }
```
### bean的实例化
此时，发现它调用了doCreateBean方法，该方法正式进入主流程了。

```
protected Object doCreateBean(final String beanName, final RootBeanDefinition mbd, Object[] args) {
        BeanWrapper instanceWrapper = null;
        if(mbd.isSingleton()) {
            instanceWrapper = (BeanWrapper)this.factoryBeanInstanceCache.remove(beanName);
        }

        if(instanceWrapper == null) {
            instanceWrapper = this.createBeanInstance(beanName, mbd, args);
        }

        final Object bean = instanceWrapper != null?instanceWrapper.getWrappedInstance():null;
        Class beanType = instanceWrapper != null?instanceWrapper.getWrappedClass():null;
        Object earlySingletonExposure = mbd.postProcessingLock;
        synchronized(mbd.postProcessingLock) {
            if(!mbd.postProcessed) {
                this.applyMergedBeanDefinitionPostProcessors(mbd, beanType, beanName);
                mbd.postProcessed = true;
            }
        }

        boolean var19 = mbd.isSingleton() && this.allowCircularReferences && this.isSingletonCurrentlyInCreation(beanName);
        if(var19) {
            if(this.logger.isDebugEnabled()) {
                this.logger.debug("Eagerly caching bean \'" + beanName + "\' to allow for resolving potential circular references");
            }

            this.addSingletonFactory(beanName, new ObjectFactory() {
                public Object getObject() throws BeansException {
                    return AbstractAutowireCapableBeanFactory.this.getEarlyBeanReference(beanName, mbd, bean);
                }
            });
        }

        Object exposedObject = bean;

        try {
            this.populateBean(beanName, mbd, instanceWrapper);
            if(exposedObject != null) {
                exposedObject = this.initializeBean(beanName, exposedObject, mbd);
            }
        } catch (Throwable var17) {
            if(var17 instanceof BeanCreationException && beanName.equals(((BeanCreationException)var17).getBeanName())) {
                throw (BeanCreationException)var17;
            }

            throw new BeanCreationException(mbd.getResourceDescription(), beanName, "Initialization of bean failed", var17);
        }

        if(var19) {
            Object ex = this.getSingleton(beanName, false);
            if(ex != null) {
                if(exposedObject == bean) {
                    exposedObject = ex;
                } else if(!this.allowRawInjectionDespiteWrapping && this.hasDependentBean(beanName)) {
                    String[] dependentBeans = this.getDependentBeans(beanName);
                    LinkedHashSet actualDependentBeans = new LinkedHashSet(dependentBeans.length);
                    String[] var12 = dependentBeans;
                    int var13 = dependentBeans.length;

                    for(int var14 = 0; var14 < var13; ++var14) {
                        String dependentBean = var12[var14];
                        if(!this.removeSingletonIfCreatedForTypeCheckOnly(dependentBean)) {
                            actualDependentBeans.add(dependentBean);
                        }
                    }

                    if(!actualDependentBeans.isEmpty()) {
                        throw new BeanCurrentlyInCreationException(beanName, "Bean with name \'" + beanName + "\' has been injected into other beans [" + StringUtils.collectionToCommaDelimitedString(actualDependentBeans) + "] in its raw version as part of a circular reference, but has eventually been " + "wrapped. This means that said other beans do not use the final version of the " + "bean. This is often the result of over-eager type matching - consider using " + "\'getBeanNamesOfType\' with the \'allowEagerInit\' flag turned off, for example.");
                    }
                }
            }
        }

        try {
            this.registerDisposableBeanIfNecessary(beanName, bean, mbd);
            return exposedObject;
        } catch (BeanDefinitionValidationException var16) {
            throw new BeanCreationException(mbd.getResourceDescription(), beanName, "Invalid destruction signature", var16);
        }
    }
```
可以看到有一个createBeanInstance方法，点击进去看看：
```
    protected BeanWrapper createBeanInstance(String beanName, RootBeanDefinition mbd, Object[] args) {
        Class beanClass = this.resolveBeanClass(mbd, beanName, new Class[0]);
        if(beanClass != null && !Modifier.isPublic(beanClass.getModifiers()) && !mbd.isNonPublicAccessAllowed()) {
            throw new BeanCreationException(mbd.getResourceDescription(), beanName, "Bean class isn\'t public, and non-public access not allowed: " + beanClass.getName());
        } else if(mbd.getFactoryMethodName() != null) {
            return this.instantiateUsingFactoryMethod(beanName, mbd, args);
        } else {
            boolean resolved = false;
            boolean autowireNecessary = false;
            if(args == null) {
                Object ctors = mbd.constructorArgumentLock;
                synchronized(mbd.constructorArgumentLock) {
                    if(mbd.resolvedConstructorOrFactoryMethod != null) {
                        resolved = true;
                        autowireNecessary = mbd.constructorArgumentsResolved;
                    }
                }
            }

            if(resolved) {
                return autowireNecessary?this.autowireConstructor(beanName, mbd, (Constructor[])null, (Object[])null):this.instantiateBean(beanName, mbd);
            } else {
                Constructor[] ctors1 = this.determineConstructorsFromBeanPostProcessors(beanClass, beanName);
                return ctors1 == null && mbd.getResolvedAutowireMode() != 3 && !mbd.hasConstructorArgumentValues() && ObjectUtils.isEmpty(args)?this.instantiateBean(beanName, mbd):this.autowireConstructor(beanName, mbd, ctors1, args);
            }
        }
    }
```
最后两行，如果ctors1不为空的话直接运行instantiateBean方法产生实例化对象。

```
   protected BeanWrapper instantiateBean(final String beanName, final RootBeanDefinition mbd) {
        try {
            Object ex;
            if(System.getSecurityManager() != null) {
                ex = AccessController.doPrivileged(new PrivilegedAction() {
                    public Object run() {
                        return AbstractAutowireCapableBeanFactory.this.getInstantiationStrategy().instantiate(mbd, beanName, AbstractAutowireCapableBeanFactory.this);
                    }
                }, this.getAccessControlContext());
            } else {
                ex = this.getInstantiationStrategy().instantiate(mbd, beanName, this);
            }

            BeanWrapperImpl bw = new BeanWrapperImpl(ex);
            this.initBeanWrapper(bw);
            return bw;
        } catch (Throwable var6) {
            throw new BeanCreationException(mbd.getResourceDescription(), beanName, "Instantiation of bean failed", var6);
        }
    }
```

到这一步了，需要调用AbstractAutowireCapableBeanFactory.this.getInstantiationStrategy().instantiate方法，点击进去发现调用了SimpleInstantiationStrategy的instantiate方法：

```
    public Object instantiate(RootBeanDefinition bd, String beanName, BeanFactory owner) {
        if(bd.getMethodOverrides().isEmpty()) {
            Object var5 = bd.constructorArgumentLock;
            Constructor constructorToUse;
            synchronized(bd.constructorArgumentLock) {
                constructorToUse = (Constructor)bd.resolvedConstructorOrFactoryMethod;
                if(constructorToUse == null) {
                    final Class clazz = bd.getBeanClass();
                    if(clazz.isInterface()) {
                        throw new BeanInstantiationException(clazz, "Specified class is an interface");
                    }

                    try {
                        if(System.getSecurityManager() != null) {
                            constructorToUse = (Constructor)AccessController.doPrivileged(new PrivilegedExceptionAction() {
                                public Constructor<?> run() throws Exception {
                                    return clazz.getDeclaredConstructor((Class[])null);
                                }
                            });
                        } else {
                            constructorToUse = clazz.getDeclaredConstructor((Class[])null);
                        }

                        bd.resolvedConstructorOrFactoryMethod = constructorToUse;
                    } catch (Exception var9) {
                        throw new BeanInstantiationException(clazz, "No default constructor found", var9);
                    }
                }
            }

            return BeanUtils.instantiateClass(constructorToUse, new Object[0]);
        } else {
            return this.instantiateWithMethodInjection(bd, beanName, owner);
        }
    }
```
最终调用BeanUtils.instantiateClass方法实例化对象：

```
   public static <T> T instantiateClass(Constructor<T> ctor, Object... args) throws BeanInstantiationException {
        Assert.notNull(ctor, "Constructor must not be null");

        try {
            ReflectionUtils.makeAccessible(ctor);
            return ctor.newInstance(args);
        } catch (InstantiationException var3) {
            throw new BeanInstantiationException(ctor.getDeclaringClass(), "Is it an abstract class?", var3);
        } catch (IllegalAccessException var4) {
            throw new BeanInstantiationException(ctor.getDeclaringClass(), "Is the constructor accessible?", var4);
        } catch (IllegalArgumentException var5) {
            throw new BeanInstantiationException(ctor.getDeclaringClass(), "Illegal arguments for constructor", var5);
        } catch (InvocationTargetException var6) {
            throw new BeanInstantiationException(ctor.getDeclaringClass(), "Constructor threw exception", var6.getTargetException());
        }
    }
```
发现ReflectionUtils.makeAccessible(ctor);这一句用来是构造函数可以访问，也就是说即使是private,protected的构造，也是可以被访问的。最后返回Object给BeanWrapper去封装，待其他扩展接口进行属性方面的扩展。
### 总结
Spring的bean的产生严格按照它的生命周期运行的，现在生成的bean的属性是还没有经过初始化的，下篇主要讲属性初始化和额外的接口实现。



