---
title: android sample 源码分析
tags:
  - android
date: 2016-12-17 15:37:22
categories: 笔记
---

以Support4Demos为例

### AndroidManifest.xml

在api demo的配置文件中声明了一个启动的主Activity

```xml
<activity android:name="Support4Demos">
	<intent-filter>
    	<action android:name="android.intent.action.MAIN" />
        <category android:name="android.intent.category.DEFAULT" />
        <category android:name="android.intent.category.LAUNCHER" />
    </intent-filter>
</activity>
```

其余的Activity声明基本如下

```xml
<activity android:name=".app.FragmentAlertDialogSupport"
          android:label="@string/fragment_alert_dialog_support">
  	<intent-filter>
    	<action android:name="android.intent.action.MAIN" />
        <category android:name="com.example.android.supportv4.SUPPORT4_SAMPLE_CODE" />
    </intent-filter>
</activity>
```

#### 一个无聊的问题

在这里有个疑惑，为什么可以有多个Activity声明为MAIN？于是对这普通得不能再普通的东西重新查了下资料，首先官方的解释是`Start as a main entry point, does not expect to receive data`，是程序的主入口点，那么主入口点又是什么，为什么可以有多个？在网上搜索也没能找到满意的答案。只能自己强行理解一番："android.intent.action.MAIN"和"android.intent.category.LAUNCHER"一起使用确定了应用启动时打开的界面，并且会在启动器界面添加多个图标，那么声明为"android.intent.action.MAIN"就可以理解为声明该Activity是独立的，并不属于上一级的Activity。这样理解好像也并不能完全解释这个疑惑，不过纠结于这也没有太大意义，只能先保留这个疑问。在主Activity中是通过action和category来过滤Activity的，所以这个还是有作用的。

### demo的主界面

官方的demo模板都是基本一样的，直接贴出源码

```java
public class Support4Demos extends ListActivity {
    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        
        Intent intent = getIntent();
        String path = intent.getStringExtra("com.example.android.apis.Path");
      	//path作为getData参数，用于下一级内容
        
        if (path == null) {
            path = "";
        }

        setListAdapter(new SimpleAdapter(this, getData(path),
                android.R.layout.simple_list_item_1, new String[] { "title" },
                new int[] { android.R.id.text1 }));
        getListView().setTextFilterEnabled(true);//使用虚拟机时，按下实体键盘可以看到过滤Activity效果
    }

    protected List<Map<String, Object>> getData(String prefix) {
        List<Map<String, Object>> myData = new ArrayList<Map<String, Object>>();

        Intent mainIntent = new Intent(Intent.ACTION_MAIN, null);//过滤条件
        mainIntent.addCategory("com.example.android.supportv4.SUPPORT4_SAMPLE_CODE");

        PackageManager pm = getPackageManager();
        List<ResolveInfo> list = pm.queryIntentActivities(mainIntent, 0);
      	//得到所有满足条件的Activity

        if (null == list)
            return myData;

        String[] prefixPath;//根据斜杠切割前缀
        String prefixWithSlash = prefix;//slash斜线
        
        if (prefix.equals("")) {
            prefixPath = null;
        } else {
            prefixPath = prefix.split("/");
            prefixWithSlash = prefix + "/";
        }
        
        int len = list.size();
        
        Map<String, Boolean> entries = new HashMap<String, Boolean>();

        for (int i = 0; i < len; i++) {
            ResolveInfo info = list.get(i);
            CharSequence labelSeq = info.loadLabel(pm);
            String label = labelSeq != null
                    ? labelSeq.toString()
                    : info.activityInfo.name;//以activity的label或name作为标题
            
            if (prefixWithSlash.length() == 0 || label.startsWith(prefixWithSlash)) {
                
                String[] labelPath = label.split("/");

                String nextLabel = prefixPath == null ? labelPath[0] :
                        labelPath[prefixPath.length];//以第一个斜杠前或前缀后的第一个内容作为标题

                if ((prefixPath != null ? prefixPath.length : 0) == labelPath.length - 1) {
                    //添加数据，此处打开的Activity是真正用于演示的Activity
                    addItem(myData, nextLabel, activityIntent(
                            info.activityInfo.applicationInfo.packageName,
                            info.activityInfo.name));
                } else {
                    //添加数据，此处打开的Activity作为浏览内容，会启动下一个Support4Demos
                  //类似于递归，如果label有'/',则再启动一个Support4Demos的Activity
                    if (entries.get(nextLabel) == null) {
                        addItem(myData, nextLabel, browseIntent(prefix.equals("") ? nextLabel : 							prefix + "/" + nextLabel));
                        entries.put(nextLabel, true);
                    }
                }
            }
        }
        Collections.sort(myData, sDisplayNameComparator);//排序      
        return myData;
    }

    private final static Comparator<Map<String, Object>> sDisplayNameComparator =
        new Comparator<Map<String, Object>>() {
        private final Collator   collator = Collator.getInstance();

        public int compare(Map<String, Object> map1, Map<String, Object> map2) {
            return collator.compare(map1.get("title"), map2.get("title"));
        }
    };

    protected Intent activityIntent(String pkg, String componentName) {//组合启动Activity的Intent
        Intent result = new Intent();
        result.setClassName(pkg, componentName);
        return result;
    }
    
    protected Intent browseIntent(String path) {//构造浏览的Activity的Intent
        Intent result = new Intent();
        result.setClass(this, Support4Demos.class);
        result.putExtra("com.example.android.apis.Path", path);
        return result;
    }

    protected void addItem(List<Map<String, Object>> data, String name, Intent intent) {
      //添加数据，标题+Intent
        Map<String, Object> temp = new HashMap<String, Object>();
        temp.put("title", name);
        temp.put("intent", intent);
        data.add(temp);
    }

    @Override
    @SuppressWarnings("unchecked")
    protected void onListItemClick(ListView l, View v, int position, long id) {
        Map<String, Object> map = (Map<String, Object>)l.getItemAtPosition(position);

        Intent intent = (Intent) map.get("intent");
        startActivity(intent);
    }
}

```

代码中最主要的部分在于getData，在这里对AndroidManifest.xml中Activity进行了分类显示，它根据Activity的label来获取标题，并且根据斜杠来分类处理，如果有需要可以打开好几级的分类界面。代码量很少，但所做的东西却很多，在我看来，这真的很神奇。

