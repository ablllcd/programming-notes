# ODOO18 开发经验

## 翻译文件

参考网址：https://www.odoo.com/documentation/saas-18.3/developer/howtos/translations.html#exporting-translatable-term

### 初始化翻译文件

1. 开启开发者模式，在setting中选择Translations目录-Export Translation目录。

2. 根据弹窗选择对应模块的po文件，从而获取一个MODULE_NAME.pot文件。

3. 将POT文件复制一份并重命名为PO文件（例如`zh_CN.po`），并在文件内填写翻译内容。

4. 将PO文件和POT文件放入对应模块的`i18n`目录下。

5. 在settiong中选择Translations目录-Languages目录，找到po文件对应的语言，点击Activate按钮激活语言。（odoo会自动读取po文件，加载到数据库中并应用）

### 更新翻译文件

1. 如果修改了odoo中文本内容或者字段名称，推荐重新导出POT文件，并且导出该模块的对应语言PO文件。PO文件中包含未修改字段的翻译内容，以及新修改的字段。

2. 当pot/po文件更改后，还是在odoo的Languages目录下，点击Update按钮更新语言,然后重启Odoo服务，并且更新module。