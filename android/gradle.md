- GRADLE
- sourcesets
  指定相关内容的路径,如:
```xml
android{
  sourceSets{
    main{
      assets.srcDirs = ['assets','src']//前边的路径就是该gradle文件的路径（即以app下的文件夹开头）
    }
  }
}
```
