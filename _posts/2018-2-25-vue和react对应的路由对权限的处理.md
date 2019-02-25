---
layout:     post
title:      React
subtitle:   权限处理
date:       2019-2-25
author:     WX
header-img: img/react.jpg
catalog: true
tags:
    - 前端
    - React
---
# 前言
很多时候我们都会有需求对应的权限处理。个人理解目前的权限有以下几种需求
    
    1.A页面需要登陆才可以访问
    2.A页面需要特定的角色才可以访问
    3.A页面的XX按钮，或者XX模块需要特定的角色才能看到   || 才能点击
    
以下我会简单的说下目前我们团队的解决方案

# 正文
    
####A页面需要登陆才可以访问

这里是入口重写的react-router-config的renderRoutes，增加了requiresAuth字段，表示该页面是否需要登陆才可以访问

        //这段话
        import React from 'react'
        import { Route, Redirect, Switch } from 'react-router-dom'
        const renderRoutes = (routes, authed, authPath = '/login', extraProps = {}, switchProps = {}) => routes ? (
          <Switch {...switchProps}>
            {routes.map((route, i) => (
              <Route
                key={route.key || i}
                path={route.path}
                exact={route.exact}
                strict={route.strict}
                render={(props) => {
                  if (!route.requiresAuth || authed || route.path === authPath) {
                    return <route.component {...props} {...extraProps} route={route} />
                  }
                        return <Redirect to={{ pathname: authPath, state: { from: props.location }}} />
                }}
              />
            ))}
          </Switch>
        ) : null
        
        export default renderRoutes   

对应的我们要修改下我们的路由

    const routes = [
        { path: '/',
            exact: true,
            component: Home,
            requiresAuth: false,
        },
        {
            path: '/login',
            component: Login,
            requiresAuth: false,
    
        },
        {
            path: '/user',
            component: User,
            requiresAuth: true, //需要登陆后才能跳转的页面
    
        },
        {
            path: '*',
            component: NotFound,
            requiresAuth: false,
        }
    ]

入口文件如下

    import React from 'react'
    import { Switch } from 'react-router-dom'
    //import { renderRoutes } from 'react-router-config'
    import renderRoutes from './utils/renderRoutes'
    import routes from './router.js'
    
    const authed = false // 如果登陆之后可以利用cookie,sessionStorage等来储存该值。对应的登陆login就在此不赘述了
    const authPath = '/login' // 默认未登录的时候返回的页面，可以自行设置
    
    const App = () => (
       <main>
          <Switch>
             {renderRoutes(routes, authed, authPath)}
          </Switch>
       </main>
    )
    export default App

 
 vue类似,而且vue有封装的路由守卫，在vuerouter里增加meta属性  meta: { requiresAuth: true }
 
    router.beforeEach((to, from, next) => {
      // 在每次路由进入之前判断requiresAuth的值，如果是true的话呢就先判断是否已登陆
    })

####需要对应的角色

其实需要对应的角色和需要登陆一回事，仔细理解，对应的requiresAuth换成roles数组或者字符串，登陆的时候获取到对应的角色，存入
cookie，在以下代码里做修改即可
    
    
        //react
      if (!route.requiresAuth || authed || route.path === authPath) {
                        return <route.component {...props} {...extraProps} route={route} />
      }
      if (route.roles.includes('xxx')) {
         return <route.component {...props} {...extraProps} route={route} />
      }
      
对应的vue的话一样，      
        
        
         router.beforeEach((to, from, next) => {
           // 在每次路由进入之前判断roles的值，如果包含的话呢就先判断是否已登陆
         })

可以在constans写一些常量，来封装路由的roles，这样可以做到一处修改处处修改。

####.A页面的XX按钮，或者XX模块需要特定的角色才能看到   || 才能点击

其实这里已经涉及到了具体的业务，可以用                                                   

    isRoles&&<button>提交</button>        

这样的方式来写，具体isRoles的获取可以用cookie 可以用 sessionStorage来做。







