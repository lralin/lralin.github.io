---
title: ant-design-pro动态菜单
date: 2019-06-29 22:50:25
updated: 2019-06-30 12:50:25
tags: 
- ant-design-pro
- react
- dva
---
### 背景
ant-design-pro升级到v4后对于动态加载菜单非常友好，只需对`src/layouts/BasicLayout.tsx`中的menuDataRender进行改造就行。[官方文档](https://pro.ant.design/docs/router-and-nav-cn#%E8%8F%9C%E5%8D%95)有说明，但是并不能直接拿来用，对于新手并不能立马实现。
<!--more-->
### 使用fetch进行动态加载菜单
1. 首先 `const [menuData=[], setMenuData] = useState([]);` menuData需要给一个默认的空数组，这样才不会导致当menuData数据还未加载成功的时候，获取面包屑映射出现异常。
   ![异常信息](https://raw.githubusercontent.com/lralin/TheFirst/master/markdown_img/ant-design-pro/ant-design-pro-1.png)
   ![代码信息](https://raw.githubusercontent.com/lralin/TheFirst/master/markdown_img/ant-design-pro/ant-design-pro-2.png)
2. 给了默认值之后，在useEffect中进行http请求，
   ```typescript
   useEffect(() => {
     fetch('/api/getMenuData')
       .then(response => response.json())
       .then(data => {
         setMenuData(data || []);
       });
   }, []);
   ```
3. 然后修改menuDataRender方法就可以实现动态加载菜单数据了。
   ```typescript
   return (
     <ProLayout
       ...
       menuDataRender={() => menuData}
       ...
     />
   );
   ```
4. 菜单格式[MenuDataItem](https://github.com/ant-design/ant-design-pro-layout/blob/56590a06434c3d0e77dbddcd2bc60827c9866706/src/typings.ts#L18)
   ```
   [
     {
       path: '/dashboard',
       name: 'dashboard',
       icon: 'dashboard',
       children: [
         {
           path: '/dashboard/analysis',
           name: 'analysis',
           exact: true,
         },
         {
           path: '/dashboard/monitor',
           name: 'monitor',
           exact: true,
         },
         {
           path: '/dashboard/workplace',
           name: 'workplace',
           exact: true,
         },
       ],
     }
     ....
   ]
   ```
   
### 使用dva dispatch来加载数据（官方推荐）
1. <div id="menuModel"></div>创建model，在src/models目录下新建menu.ts，内容如下。
```typescript
import { getMenuData } from '@/services/menu';
import { Effect } from 'dva';
import { Reducer } from 'redux';
import {MenuDataItem} from "@ant-design/pro-layout";

export interface MenuModelState {
  menuData?: MenuDataItem[];
}

export interface MenuModelType {
  namespace: 'menu';
  state: MenuModelState;
  effects: {
    getMenuData: Effect;
  };
  reducers: {
    saveMenu: Reducer<MenuModelState>;
  };
}
const MenuModel: MenuModelType = {
  namespace: 'menu',
  state: {
    menuData: [],
  },

  effects: {
    *getMenuData({ payload, callback }, { call, put }) {
      const response = yield call(getMenuData);
      yield put({
        type: 'saveMenu',
        payload: response,
      });
    },
  },

  reducers: {
    saveMenu(state, action) {
      return {
        ...state,
        menuData: action.payload || [],
      };
    },
  },
};
export default MenuModel;
```
2. 创建service，在src/services目录下，创建menu.ts，内容如下。
```typescript
import request from '@/utils/request';

export async function getMenuData(): Promise<any> {
  return request('/menu/menuData');
}
```
3. 创建mock数据，我这里就不重新创建menu单独的文件了，直接把mock数据写在了route.ts文件里面。
```typescript
export default {

  ...
  
  'Get /menu/menuData': [
    {
      path: '/dashboard',
      name: 'dashboard',
      icon: 'dashboard',
      children: [
        {
          path: '/dashboard/analysis',
          name: 'analysis',
          exact: true,
        },
        {
          path: '/dashboard/monitor',
          name: 'monitor',
          exact: true,
        },
        {
          path: '/dashboard/workplace',
          name: 'workplace',
          exact: true,
        },
      ],
    }
  ],
  
  ...
};

```

4. 现在我们model已经准备好了，就剩model和view关联的问题了，那么BasicLayout.tsx如何加载后台数据呢？。初学dva和react的我，对于connect掌握的不是很熟练，于是我就先根据currentUser加载方式写了一遍，其中需要注意的东西是非常的多。
    - 首先找到connect的mapStateToProps，在BasicLayout.tsx的末尾。如果直接加上menu是会报错的，因为ConnectState中并没有menu类型。
      ```
      export default connect(({ global, settings, menu}: ConnectState) => ({
      collapsed: global.collapsed,
      settings,
      menuData:menu.menuData,
      }))(BasicLayout);
      ```
    - 所以我们找到ConnectState增加`menu: MenuModelState;`其中 **menu** 就是我们刚刚[model](#menuModel)中的namespace，如果此处名字和namespace不对应会导致数据加载不上了，非常重要；**MenuModelState** 也是在menu的[model](#menuModel)中定义的。
    ```
    export interface ConnectState {
      global: GlobalModelState;
      loading: Loading;
      settings: SettingModelState;
      user: UserModelState;
      menu: MenuModelState;
    }
    ```
    - ConnectState和mapStateToProps设置好之后，数据就到了props中，由于ProLayout中默认有menuData，所以如果props中传入了menuData，那么菜单就会用menuData的数据，而不是route的数据，至此动态菜单渲染成功。

### 参考资料
- [react小书](http://huziketang.mangojuice.top/books/react/lesson36)帮助我入门了react还让我理解了react-redux，这有利于我学习ant-design-pro中的dva。

- [ant-design-pro中ProLayout的api](https://github.com/ant-design/ant-design-pro-layout#prolayout)

- 文档中的model是参考[官方issues](https://github.com/ant-design/ant-design-pro/issues/4358)写的
