# Unity 编译DLL

## 关系链
Scripting Runtime Version ---> Assembly Definition ---> UGlue.csproj: <TargetFrameworkVersion>v3.5</TargetFrameworkVersion>
即，每次双击脚本，Unity会参考Runtime Version, 配置到scproj项目属性，右键csproj项目，生成的DLL只能运行在Runtime Version以上版本。

鉴于上述关系，若于 4.x 的项目中开发DLL，若要发布到 3.5 平台， 则需要设置Scripting Runtime Version 为3.5。
设置完后会发现一堆报错，暂时不用管unity editor下的报错，只需要在相应的csproj项目右键重新生成项目，过程中出现的“语法需要C# 7.0支持”之类的
，根据系统提示，设置C#语言到相应版本即可。所有错误信息处理完毕即可生成相应的DLL，之后再将项目切换回去就行了。

> 考虑下是否可以将csprj 项目文件备份一份到其他目录，后续需要编译的时候直接打开这个csprj编译即可，避免侵入unity

> 根据关系链，想要做到直接修改csproj，然后编译，是行不通的，每次重新打开，设置都会被覆盖掉。

## DLL警告
the plugin references at least one UnityEngine module：貌似是重复引用同一个模块的问题，去除重复引用试试

## API Compatibility Level 
影响编译完的DLL的运行情况，选择2.0，则基本支持所有的unity项目，选择4.0，会调用到一些4.0的内容，在一些 runtime version 3.5的项目里可能出错。
