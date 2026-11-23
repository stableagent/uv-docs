# metadata مربوط به workspace {#workspace-metadata}

`uv workspace metadata` اطلاعاتی را که uv دربارهٔ workspace یا script مربوط به PEP 723 شما دارد،
به‌صورت JSON صادر می‌کند تا toolهای دیگر بتوانند از آن استفاده کنند. اگر به اطلاعات موجود در یک
`uv.lock` یا lockfile مربوط به script دسترسی می‌خواهید، خروجی این دستور را ترجیح دهید: قالب
lockfile تضمین‌شده نیست که پایدار بماند. برای درخواست metadata مربوط به یک script،
`--script path/to/script.py` را ارسال کنید.

برای نصب packageهای انتخاب‌شده پیش از جمع‌آوری اطلاعات مالکیت module، `--sync` را ارسال کنید.
همگام‌سازی به‌طور پیش‌فرض packageهای نصب‌شدهٔ نامرتبط را حفظ می‌کند؛ برای حذف آن‌ها `--exact` را
اضافه کنید.

ساختار اصلی، فیلد "resolution" است که گراف dependency را با نسخه‌های دقیق package که یک `uv.lock`
رمزگذاری می‌کند، در بر می‌گیرد.

یال‌های گراف، `dependencies`ای هستند که هر node تعریف می‌کند. این‌ها نیز باید هنگام نصب node نصب
شوند، همراه با `dependencies` آن‌ها به‌صورت بازگشتی. چرخه‌ها در این گراف عادی هستند. هر ورودی
dependency شامل یک `id` برای node‌ای است که به آن اشاره می‌کند و یک `marker` اختیاری که
[مشخص می‌کند dependency روی چه platformهایی لازم است](https://packaging.python.org/en/latest/specifications/dependency-specifiers/#dependency-specifiers)
(اگر marker‌ای نباشد، dependency همیشه لازم است).

nodeهای مشتق از package در گراف، به‌صورت یکتا با `name` و `version` و `source` و `kind` مربوط به
package شناسایی می‌شوند. nodeهای script و workspace با مسیرشان شناسایی می‌شوند. nodeهای group
مربوط به dependency در ریشهٔ workspace با نام group و مسیر workspace شناسایی می‌شوند. همهٔ idهای
node باید به‌عنوان مبهم (opaque) در نظر گرفته شوند.

۵ نوع node در گراف وجود دارد:

- `"script"` -- یک script مربوط به PEP 723 و dependencyهای مستقیم آن
- `"workspace"` -- یک ریشهٔ workspace و groupهای dependency مختص workspace آن
- `"package"` -- خود package
- `{ "extra": "extraname" }` -- یک extra که package تعریف می‌کند
- `{ "group": "groupname" }` -- یک dependency group که یک package یا ریشهٔ workspace تعریف می‌کند

(در آینده nodeهای "build" را برای dependencyهای
[environmentهای build](../../concepts/projects/config.md#build-isolation) اضافه خواهیم کرد.)

اگر می‌خواهید `mypackage` را نصب کنید، node با `"kind": "package"` آن را پیدا کنید. این node
همچنین اطلاعاتی دربارهٔ sdist آن، wheelهای آن، extraهای آن (`optional_dependencies`) و dependency
groupهای آن (`dependency_groups`) شامل می‌شود.

اگر می‌خواهید `mypackage[myextra]` را نصب کنید، node با `"kind": { "extra": "myextra" }` را برای
`mypackage` پیدا کنید (این node همیشه به `mypackage` وابسته خواهد بود). اگر می‌خواهید
`mypackage[extra1, extra2]` را نصب کنید، دو node مربوط به `mypackage[extra1]` و
`mypackage[extra2]` را پیدا کنید.

اگر می‌خواهید dependency group با نام `mypackage:mygroup` را نصب کنید، node با
`"kind": { "group": "mygroup" }` را برای `mypackage` پیدا کنید (این node به `mypackage` وابسته
_نخواهد_ بود، چون dependency groupها فقط فهرست‌هایی از چیزهایی هستند که ممکن است هنگام کار روی
خود package بخواهید).

اگر ریشهٔ workspace dependency groupهایی تعریف کند اما خودش یک package نباشد، node با نام
`"workspace"` آن، idهای node مربوط به group را از طریق `dependency_groups` ارائه می‌دهد.

## packageهای نصب‌شده {#installed-packages}

وقتی یک environment وجود داشته باشد، `environment.packages` فهرستی از distributionهای نصب‌شدهٔ آن
است، از جمله مواردی که در lockfile نیستند. هر ورودی شامل نام نصب‌شده، نسخه، مسیر metadata و وضعیت
editable است.

یک distribution نصب‌شده بر اساس نام نرمال‌شده با یک package در resolution انتخاب‌شده تطبیق داده
می‌شود. یک ورودی تطبیق‌داده‌شده سپس از ID مربوط به node مربوط به resolution در آن package استفاده
می‌کند. فیلد `version` در ورودی، نسخهٔ واقعی نصب‌شده را گزارش می‌دهد؛ IDهای node باید به‌عنوان
مبهم (opaque) در نظر گرفته شوند.

فیلد `modules` نام‌های module قابل import را به شیءهایی نگاشت می‌کند که آرایهٔ `owners` آن‌ها شامل
ارجاع‌های `package_id` به `environment.packages` است. IDهای packageهای خارج از resolution
انتخاب‌شده در `resolution` ظاهر نمی‌شوند. وقتی map خالی باشد، حذف می‌شود.

## برخورد با چند نسخه از یک package {#handling-multiple-versions-of-a-package}

دو نسخه از یک package نمی‌توانند در یک Python environment نصب شوند، اما گراف dependency ممکن است
همچنان چند نسخه از یک package را شامل شود. این موضوع به دو دلیل متفاوت می‌تواند رخ دهد.

راه اول این است که
[platformهای مختلف](https://packaging.python.org/en/latest/specifications/dependency-specifiers/#dependency-specifiers)
requirementهای متناقضی داشته باشند که استفاده از نسخه‌های متفاوت یک package را اجبار می‌کنند.

راه دوم وقتی است که یک workspace دارای
[تعارض‌ها](../../concepts/resolution.md#conflicting-dependencies) باشد؛ به این معنا که بعضی از
اعضای workspace یا extraهای آن‌ها با یکدیگر ناسازگارند و فقط یکی از آن‌ها می‌تواند در یک زمان نصب
شود. اطلاعات مربوط به تعارض‌ها را می‌توان در فیلد سطح بالای `conflicts` پیدا کرد.

تضمین مشخصی که ما ارائه می‌دهیم این است که **برای هر انتخاب مشخص از
[markerها](https://packaging.python.org/en/latest/specifications/dependency-specifiers/#dependency-specifiers)،
اگر مجموعه‌ای از packageها را برای نصب انتخاب کنید که هیچ
[تعارضی](../../concepts/resolution.md#conflicting-dependencies) نداشته باشد، مجموعهٔ حاصل از
packageها برای نصب، چند نسخه از یک package نخواهد داشت**.

اگر فقط می‌خواهید «هر نسخه از pydantic که این workspace استفاده می‌کند» را بگیرید، آزادید که در
فهرست nodeها پیمایش کنید و هر نمونه را جمع کنید. اما اگر می‌خواهید به‌طور خاص گراف را تحلیل کنید و
resolutionهای واقعی را بگیرید، احتمالاً باید به `conflicts` مراجعه کنید و باید بدانید چگونه
`marker`ها را برای یک platform مشخص حل کنید.

بهترین راه برای جلوگیری از اشتباه هنگام کار با چند نسخه از یک package این است که پرس‌وجوهای خود
در گراف dependency را بر عملیات روی ریشهٔ workspace، اعضای workspace یا script درخواستی متمرکز
کنید. این‌ها نقطه‌های ورود طبیعی به گراف هستند و می‌توانند پاسخ‌های منسجمی برای عملیات‌هایی مثل
«نصب group با نام `dev` در workspace»، «نصب `member1` و `member2[extra]`» یا «نصب dependencyهای
اعلام‌شدهٔ این script» ارائه دهند.

به بیان دیگر، وقتی ممکن است _باید از پیمایش روی شیء `resolution` برای پیدا کردن یک node پرهیز
کنید_. فقط با استفاده از idهایی که بخش دیگری از metadata ارائه کرده است، به `resolution` مثل یک
map دسترسی پیدا کنید. برای یک workspace، ریشهٔ workspace برابر `workspace.id` است و نقطه‌های ورود
package در آرایهٔ `members` فهرست شده‌اند. برای یک script، id اولیه برابر `script.id` است. از آنجا
می‌توانید با دنبال کردن یال‌های dependency، packageهای دیگر را به‌صورت بازگشتی کشف کنید.

پس به‌جای اینکه مستقیماً سعی کنید node‌ای برای anyio در گراف dependency پیدا کنید، باید تصمیم
بگیرید که کدام عضو (یا اعضای) workspace را می‌خواهید تحلیل کنید، گویی قرار است نصب شوند. هنگام
پیمایش `dependencies` چیزهایی که می‌خواهید نصب کنید، ممکن است به یک نمونه از anyio برسید که همان
نمونه‌ای است که باید استفاده کنید. اگر به چند نمونه از anyio برسید، یعنی مجموعه‌ای متناقض از
چیزهایی را برای نصب انتخاب کرده‌اید که uv هرگز آن‌ها را انتخاب نمی‌کند.

پس اگر می‌خواستید مثلاً نصب dependency group با نام `dev` مربوط به عضو workspace با نام
`mypackage` را تحلیل کنید، چیزی شبیه به این خواهد بود:

```python
member = find_by_name(metadata.members, "mypackage")
member_node = metadata.resolution[member.id]
group = find_by_name(member_node.dependency_groups, "dev")
group_node = metadata.resolution[group.id]
visit(metadata, [group_node])
```

برای یک dependency group که روی ریشهٔ workspace تعریف شده است، آن را از طریق node مربوط به
workspace جستجو کنید:

```python
workspace_node = metadata.resolution[metadata.workspace.id]
group = find_by_name(workspace_node.dependency_groups, "dev")
group_node = metadata.resolution[group.id]
visit(metadata, [group_node])
```

اگر می‌خواستید دو عضو خاص از workspace را که با هم نصب شده‌اند تحلیل کنید، چیزی شبیه به این خواهد
بود:

```python
to_analyze = []
for member_name in ["package1", "package2"]:
    member = find_by_name(metadata.members, member_name)
    member_node = metadata.resolution[member.id]
    to_analyze.append(member_node)
visit(metadata, to_analyze)
```

برای یک script، دقیقاً به همان روش از node مربوط به resolution آن شروع کنید:

```python
script_node = metadata.resolution[metadata.script.id]
visit(metadata, [script_node])
```

که در آن `visit` الگوریتم پیمایش گراف مورد علاقهٔ شماست، مثل جستجوی عمق‌اول (depth-first-search):

```python
def visit(metadata: UvMetadata, to_analyze: list[Node]):
    visited = set()
    while len(to_analyze) > 0:
        node = to_analyze.pop()

        # Handle cycles by avoiding revisiting nodes
        if node.id in visited:
            continue
        visited.add(node.id)

        # We also need to analyze its dependencies
        for dependency in node.dependencies:
            # Only follow edges if they satisfy the desired platform's markers
            if dependency.marker and not satisfies(platform, dependency.marker):
                continue
            to_analyze.append(metadata.resolution[dependency.id])

        # Analyze any package node we encounter
        if node.kind == "package":
            print(node.name, node.version, node.source)
```

## Schema {#schema}

برای درخواست schema با نام `preview`، از `uv workspace metadata --output-version preview` استفاده
کنید. در حال حاضر این تنها نسخهٔ پشتیبانی‌شده است. اگر فلگ را حذف کنید یا نسخهٔ
پشتیبانی‌نشده‌ای را درخواست کنید، uv جدیدترین schema‌ای را که پشتیبانی می‌کند برمی‌گرداند. فیلد
`schema.version` نسخهٔ برگردانده‌شده را مشخص می‌کند.

یک JSON schema کامل برای این قالب، وقتی قالب نهایی شد، ارائه خواهد شد.

در ادامه یک مثال حاشیه‌نویسی‌شده و خوانا برای انسان آمده است:

```js
{
  // Information about the schema of this output
  "schema": {
    // The version of this output, currently "preview"
    "version": "preview"
  },
  // The directory the uv.lock can be found in
  "workspace_root": "/workspace",
  // Information about the environment, available when an environment exists or `--sync` is used
  "environment": {
    // The absolute path to the environment root
    "root": "/workspace/.venv",
    // Information about the Python interpreter in the environment
    "python": {
      // The absolute path to the Python executable
      "path": "/workspace/.venv/bin/python",
      // The full Python version
      "version": "3.12.12",
      // The Python implementation name
      "implementation": "cpython"
    },
    // Installed distributions, keyed by opaque node IDs
    "packages": {
      "idna==3.10@unmanaged+/workspace/.venv/lib/python3.12/site-packages/idna-3.10.dist-info": {
        "name": "idna",
        "version": "3.10",
        "path": "/workspace/.venv/lib/python3.12/site-packages/idna-3.10.dist-info",
        "editable": false
      }
    }
  },
  // Information about the script target, only present with `--script`.
  // Workspace metadata uses `workspace` and `members` below as graph entry-points instead.
  "script": {
    // The absolute path to the script
    "path": "/workspace/script.py",
    // The id of the script's node in the `resolution` map below
    "id": "script+/workspace/script.py"
  },
  // Information about the workspace target, omitted when `--script` is used.
  "workspace": {
    // The absolute path to the workspace root
    "path": "/workspace",
    // The id of the workspace's node in the `resolution` map below
    "id": "workspace+/workspace"
  },
  // Any requirements on the python version this workspace has
  //
  // `marker` fields all have this as an implicit constraint that is omitted for cleanliness
  "requires_python": ">=3.12",
  // Module owners refer to entries in `environment.packages`.
  // Matched package IDs also refer to nodes in `resolution`.
  "modules": {
    "idna": {
      "owners": [
        {
          "package_id": "idna==3.10@unmanaged+/workspace/.venv/lib/python3.12/site-packages/idna-3.10.dist-info"
        }
      ]
    }
  },
  // A list of workspace members
  "members": [
    {
      // The name of the package
      "name": "mypackage",
      // The directory that contains its pyproject.toml
      "path": "/workspace/packages/mypackage",
      // The id of this package's info in the `resolution` map below
      "id": "mypackage==0.1.0@editable+/workspace/packages/mypackage"
    },
  ],
  // A list-of-sets of workspace items that are mutually-exclusive to install,
  // presumably because they need to install different versions of the same package.
  //
  // Any attempt to install two things that belong to the same set must be rejected.
  //
  // There are 3 kinds of item:
  //
  // * Project -- "kind": "project"
  // * Extra   -- "kind": { "extra": "extraname" }
  // * Group   -- "kind": { "group": "groupname" }
  "conflicts": {
    "sets": [
      {
        "items": [
          {
            "package": "mypackage",
            "kind": { "extra": "myextra" }
            "id": "mypackage[myextra]==0.1.0@editable+/workspace/packages/mypackage",
          }
          {
            "package": "mypackage",
            "kind": { "group": "mygroup" }
            "id": "mypackage:mygroup==0.1.0@editable+/workspace/packages/mypackage",
          }
        ]
      }
    ]
  }
  // Resolved information about packages and dependencies.
  //
  // Each entry in this map is a node in the dependency graph. There are currently
  // 5 kinds of node in the dependency graph, although more are planned in the future.
  //
  // * Scripts  -- "kind": "script"
  // * Workspaces -- "kind": "workspace"
  // * Packages -- "kind": "package"
  // * Extras   -- "kind": { "extra": "extraname" }
  // * Groups   -- "kind": { "group": "groupname" }
  //
  // Package nodes contain most of the metadata, while other nodes are mostly just a list
  // of dependencies. The different kinds of node are included like this to encourage correct
  // analysis of the graph. For instance, a node for `mypackage[someextra]` always depends on
  // `mypackage`, while `mypackage:somegroup` does not (because dependency-groups are just a
  // list of packages you might want to install while working on `mypackage`). Sugars like
  // `mypackage[extra1, extra2]` are decomposed into separate dependencies on `mypackage[extra1]`
  // and `mypackage[extra2]`.
  //
  // The ids used here are human-readable but should be handled as opaque (the nodes contain
  // the same information in a more convenient form).
  "resolution": {

    // The script node is present when metadata was requested with `--script`. Its dependencies
    // are the direct requirements declared by the script.
    "script+/workspace/script.py": {
      "kind": "script",
      "path": "/workspace/script.py",
      "dependencies": [
        {
          "id": "iniconfig==2.0.0@registry+https://pypi.org/simple"
        }
      ]
    },

    // The workspace node owns metadata defined directly on the workspace root.
    "workspace+/workspace": {
      "kind": "workspace",
      "path": "/workspace",
      "dependencies": [],
      "dependency_groups": [
        {
          "name": "dev",
          "id": "workspace+/workspace:dev"
        }
      ]
    },

    // This node is a dependency group defined on the non-package workspace root.
    "workspace+/workspace:dev": {
      "kind": { "group": "dev" },
      "path": "/workspace",
      "dependencies": [
        {
          "id": "iniconfig==2.0.0@registry+https://pypi.org/simple"
        }
      ]
    },

    // This node is a workspace member
    "mypackage==0.1.0@editable+/workspace/packages/mypackage": {
      // The name of the package
      "name": "mypackage",
      // The version of the package (this may be missing, as source trees do not need versions)
      "version": "0.1.0",
      // The source of the package, in this case it's an editable whose path relative to the
      // `workspace_root` is `./packages/mypackage`
      "source": {
        "editable": "/workspace/packages/mypackage"
      },
      // The kind of the node, in this case "package" (see the docs on `resolution` above for details)
      "kind": "package",
      // The dependencies that must be installed to also install this node into an environment
      "dependencies": [
        {
          // The id of the node to lookup for details
          "id": "iniconfig==2.0.0@registry+https://pypi.org/simple"
          "marker": "marker": "sys_platform == 'linux'"
        }
      ],
      // The extras that this package defines
      "optional_dependencies": [
        {
          "name": "myextra",
          "id": "mypackage[myextra]==0.1.0@editable+/workspace/packages/mypackage"
        }
      ],
      // The dependency groups this package defines
      "dependency_groups": [
        {
          "name": "mygroup",
          "id": "mypackage:mygroup==0.1.0@editable+/workspace/packages/mypackage"
        }
      ]
    },

    // This node is an extra on a workspace member
    "mypackage[myextra]==0.1.0@editable+/workspace/packages/mypackage": {
      // These fields will match the package node above
      "name": "mypackage",
      "version": "0.1.0",
      "source": {
        "editable": "/workspace/packages/mypackage"
      },
      // But these two will differ from the package node above
      "kind": { "extra": "myextra" },
      "dependencies": [
        {
          "id": "mypackage==0.1.0@editable+/workspace/packages/mypackage"
        }
        {
          "id": "anyio==2.0.0@registry+https://pypi.org/simple"
        }
      ]
    },

    // This node is a dependency-group on a workspace member
    "mypackage:mygroup==0.1.0@editable+/workspace/packages/mypackage": {
      // These fields will match the package node above
      "name": "mypackage",
      "version": "0.1.0",
      "source": {
        "editable": "/workspace/packages/mypackage"
      },
      // But these two will differ from the package node above
      "kind": { "extra": "myextra" },
      "dependencies": [
        {
          "id": "anyio==1.0.0@registry+https://pypi.org/simple"
        }
      ]
    },

    // This node is a package on pypi
    "iniconfig==2.0.0@registry+https://pypi.org/simple": {
      "name": "iniconfig",
      "version": "2.0.0",
      // registry sources look like this
      "source": {
        "registry": {
          "url": "https://pypi.org/simple"
        }
      },
      "kind": "package",
      "dependencies": [],
      // Details on the package's source distribution
      "sdist": {
        // May alternatively be `path`
        "url": "https://files.pythonhosted.org/packages/d7/4b/cbd8e699e64a6f16ca3a8220661b5f83792b3017d0f79807cb8708d33913/iniconfig-2.0.0.tar.gz",
        "hashes": {
          "sha256": "2d91e135bf72d31a410b17c16da610a82cb55f6b0477d1a902134b24a455b8b3"
        },
        "size": 4646,
        "upload_time": "2023-01-07T11:08:11.254Z"
      },
      // The wheels we found for this package
      "wheels": [
        {
          // May alternatively be `path`
          "url": "https://files.pythonhosted.org/packages/ef/a6/62565a6e1cf69e10f5727360368e451d4b7f58beeac6173dc9db836a5b46/iniconfig-2.0.0-py3-none-any.whl",
          "hashes": {
            "sha256": "b6a85871a79d2e3b22d2d1b94ac2824226a63c6b741c88f7ae975f18b6778374"
          },
          "size": 5892,
          "upload_time": "2023-01-07T11:08:09.864Z",
          // Parsing this name is how you know what platform a wheel supports
          "filename": "iniconfig-2.0.0-py3-none-any.whl"
        }
      ]
    }

    // ...and so on
    "anyio==1.0.0@registry+https://pypi.org/simple": { ... }
    "anyio==2.0.0@registry+https://pypi.org/simple": { ... }
  }
}
```
