---
layout:     post
title:      "MyBatis-参数封装过程"
subtitle:   "参数封装源码分析"
date:       2019-01-22 23:10
author:     "LQFGH"
header-img: "img/post-bg-halting.jpg"
header-mask: 0.3
catalog:    true
tags:
  - MyBatis
  - Source Code
---

**查询抽象方法**
```java
Employee selectByIdAndName(@Param("id") Integer id,@Param("lastName") String lastName);
```


**解析参数的类**
```java

package org.apache.ibatis.reflection;

//省略导包

public class ParamNameResolver {

  private static final String GENERIC_NAME_PREFIX = "param";
  private static final String PARAMETER_CLASS = "java.lang.reflect.Parameter";
  private static Method GET_NAME = null;
  private static Method GET_PARAMS = null;

  static {
    try {
      Class<?> paramClass = Resources.classForName(PARAMETER_CLASS);
      GET_NAME = paramClass.getMethod("getName");
      GET_PARAMS = Method.class.getMethod("getParameters");
    } catch (Exception e) {
      // ignore
    }
  }

  private final SortedMap<Integer, String> names;

  private boolean hasParamAnnotation;

  public ParamNameResolver(Configuration config, Method method) {
    final Class<?>[] paramTypes = method.getParameterTypes();
    final Annotation[][] paramAnnotations = method.getParameterAnnotations();
    final SortedMap<Integer, String> map = new TreeMap<Integer, String>();
    int paramCount = paramAnnotations.length;
    // get names from @Param annotations
    for (int paramIndex = 0; paramIndex < paramCount; paramIndex++) {
      if (isSpecialParameter(paramTypes[paramIndex])) {
        // skip special parameters
        continue;
      }
      String name = null;
      for (Annotation annotation : paramAnnotations[paramIndex]) {
        if (annotation instanceof Param) {
          hasParamAnnotation = true;
          name = ((Param) annotation).value();
          break;
        }
      }
      if (name == null) {
        // @Param was not specified.
        if (config.isUseActualParamName()) {
          name = getActualParamName(method, paramIndex);
        }
        if (name == null) {
          // use the parameter index as the name ("0", "1", ...)
          // gcode issue #71
          name = String.valueOf(map.size());
        }
      }
      map.put(paramIndex, name);
    }
    names = Collections.unmodifiableSortedMap(map);
  }

  private String getActualParamName(Method method, int paramIndex) {
    if (GET_PARAMS == null) {
      return null;
    }
    try {
      Object[] params = (Object[]) GET_PARAMS.invoke(method);
      return (String) GET_NAME.invoke(params[paramIndex]);
    } catch (Exception e) {
      throw new ReflectionException("Error occurred when invoking Method#getParameters().", e);
    }
  }

  private static boolean isSpecialParameter(Class<?> clazz) {
    return RowBounds.class.isAssignableFrom(clazz) || ResultHandler.class.isAssignableFrom(clazz);
  }

  /**
   * Returns parameter names referenced by SQL providers.
   */
  public String[] getNames() {
    return names.values().toArray(new String[0]);
  }


  public Object getNamedParams(Object[] args) {
    final int paramCount = names.size();
    if (args == null || paramCount == 0) {
      return null;
    } else if (!hasParamAnnotation && paramCount == 1) {
      return args[names.firstKey()];
    } else {
      final Map<String, Object> param = new ParamMap<Object>();
      int i = 0;
      for (Map.Entry<Integer, String> entry : names.entrySet()) {
        param.put(entry.getValue(), args[entry.getKey()]);
        // add generic param names (param1, param2, ...)
        final String genericParamName = GENERIC_NAME_PREFIX + String.valueOf(i + 1);
        // ensure not to overwrite parameter named with @Param
        if (!names.containsValue(genericParamName)) {
          param.put(genericParamName, args[entry.getKey()]);
        }
        i++;
      }
      return param;
    }
  }
}
```