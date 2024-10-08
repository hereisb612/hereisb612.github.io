# SpringMVC

# Configurations

**CharacterEncodingFilter**: set encoding like UTF-8 or else

forceResponseEncoding: set ResponseEncoding

**HiddenHttpMethodFilter**: used for Restful style. By using this filter, can set methods in _methods to get post put delete for different actions but form always using POST.

```
<filter>
    <filter-name>CharacterEncodingFilter</filter-name>
    <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
    <init-param>
        <param-name>encoding</param-name>
        <param-value>UTF-8</param-value>
    </init-param>
    <init-param>
        <param-name>forceResponseEncoding</param-name>
        <param-value>true</param-value>
    </init-param>
</filter>
<filter-mapping>
    <filter-name>CharacterEncodingFilter</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>

<filter>
    <filter-name>HiddenHttpMethodFilter</filter-name>
    <filter-class>org.springframework.web.filter.HiddenHttpMethodFilter</filter-class>
</filter>
<filter-mapping>
    <filter-name>HiddenHttpMethodFilter</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```

**contextConfigLocation**: link to spring config files.

```
<servlet>
    <servlet-name>DispatcherServlet</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <init-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>classpath:SpringMVC.xml</param-value>
    </init-param>
    <load-on-startup>1</load-on-startup>
</servlet>
<servlet-mapping>
    <servlet-name>DispatcherServlet</servlet-name>
    <url-pattern>/</url-pattern>
</servlet-mapping>
```



# Scope

## ModelAndView (Recommended)

> has 2 basis functions
>
> Model: share data in different scope like request and session
>
> view: set view to control the re-direct to other pages



### Usage

*functions need to return a ModelAndView object*

```java
ModelAndView mav = new ModelAndView();

mav.addObject(key, value); // share data

mav.setViewName(viewName);

return mav; // same like return a String to thymeleaf
```



## Model / Map / ModelMap (Optional)

> the split of ModelAndView



### Usage

*Need to wirte like 'Model model' in the formla parameter and the Spring framework will inject the Model object automatically by reflection. Additionally, donnot need to return a Model object, can return view name as String as normal.*



# View in SpringMVC

## View

> handle by view templates engine configed by user in configuration files like thymeleaf.



## InternalResourceView

> Forward. One-round request.

``````
return "forward:/test"
``````



## RedirectView

> Re-direct. Two-round request.

``````
return "redirect:/test"
``````



## View Controller

> if the only function(action) of a method in controller is return a view name, such as `public String index(){return index;}`, can replace this function in springXML.xml as `<mvc:view-controller path="/" view-name="index"/>`. More simple and readable.



**if wirte view-controller in xml files, all the controller about view will disable. **

**But by adding this tag   ``````<mvc:annotation-driven/> ``````  can fix this problem.**



# Restful

- get means select
- post means insert into
- put means alter (update)
- delete means remove



# HttpMessageConverter

> provides 2 annotations and 2 class
>
> @RequestBody + RequestEntity -> used for request
>
> @ResponseBody + ResponseEntity -> used for response



## @RequestBody

can use this annotation at formal variable place, and then the data in this request will be convert to a java object automatically. Like:

*by the way, the data means that the method of form only can be POST because the data need to in the request body*

*moreover, the java object converted from this annotation is only about the real data. IF wanna have request head or something else, need to use RequestEntity.*

```
@RequestMapping("/RequestBodyTest")
public String RequestBodyTest(@RequestBody String body) {
    System.out.println(body);
    return "success";
}
```



## RequestEntity

this Java class can convert all the request, and also have methods to get headers or other informations.

```
@RequestMapping("/RequestEntityTest")
public String RequestEntityTest(RequestEntity<String> body) {
    System.out.println(body.getHeaders());
    System.out.println(body.getBody());
    return "success";
}
```



## @ResponseBody

this annotation can set data in Return as the data in response body.

If have this annotation, data after return will be set to the context in the page. And if donot have this annotion, data after return will be seem as ViewName.
