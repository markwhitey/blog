---
title: PowerDesigner数据库设计基于Excel导入导出总结
mathjax: false
date: 2020-02-06 22:12:44
tags:
- 数据仓库
categories:
- Big Data
- Data Warehouse
top:
photo:
---



{% cq %}

在数据库建模中会用到Powerdesigner软件进行表结构的设计，有时候我们需要将Excel里面的表结构导入到Powerdesigner中生成PDM模型文件，或者将Powerdesigner中已有的PDM模型导出生成Excel文档；我们可以通过Powerdesigner的脚本定制功能，来实现Excel的导入导出。

{% endcq %}

<!-- more -->

<br>

# PDM导出excel

---

**在PowerDesigner 中 ctrl+shift+x 弹出执行脚本界面，输入如下代码就会生成 Excel(第一个 Sheet 页上是所有表的目录列表，每个表都会新建一个 Sheet 页)**

```vb
'******************************************************************************
'* File:     Exported_Excel_page.vbs
'* Purpose:  分目录递归，查找当前PDM下所有表，并导出Excel,添加目录页

'******************************************************************************

Option Explicit
ValidationMode = True
InteractiveMode = im_Batch

'-----------------------------------------------------------------------------
' 主函数
'-----------------------------------------------------------------------------
' 获取当前活动模型
Dim mdl ' 当前的模型
Set mdl = ActiveModel
Dim EXCEL,catalog,sheet,catalogNum,rowsNum,linkNum
rowsNum = 1
catalogNum = 1
linkNum = 1

If (mdl Is Nothing) Then
    MsgBox "There is no Active Model"
Else
    SetCatalog
    ListObjects(mdl)
End If

'----------------------------------------------------------------------------------------------
' 子过程，用于扫描当前包并从当前包中打印对象的信息，然后对当前包的所有子包再次调用相同的子过程
'----------------------------------------------------------------------------------------------
Private Sub ListObjects(fldr)
    output "Scanning " & fldr.code
    Dim obj ' 运行对象
    For Each obj In fldr.children
        ' 调用子过程来打印对象上的信息
        DescribeObject obj
    Next
    ' 进入子包
    Dim f ' 运行文件夹
    For Each f In fldr.Packages
        '调用子程序扫描子程序包
        ListObjects f
    Next
End Sub

'-----------------------------------------------------------------------------
' 子过程，用于在输出中打印当前对象的信息
'-----------------------------------------------------------------------------
Private Sub DescribeObject(CurrentObject)
    if not CurrentObject.Iskindof(cls_NamedObject) then exit sub
    if CurrentObject.Iskindof(cls_Table) then 
        AddSheet CurrentObject.code
        ExportTable CurrentObject, sheet
        ExportCatalog CurrentObject
    else
        output "Found "+CurrentObject.ClassName+" """+CurrentObject.Name+""", Created by "+CurrentObject.Creator+" On "+Cstr(CurrentObject.CreationDate)   
    End if
End Sub

'----------------------------------------------------------------------------------------------
' 设置Excel的sheet页
'----------------------------------------------------------------------------------------------
Sub SetExcel()
    Set EXCEL= CreateObject("Excel.Application")

    ' 使Excel通过应用程序对象可见。
    EXCEL.Visible = True
    EXCEL.workbooks.add(-4167)'添加工作表
    EXCEL.workbooks(1).sheets(1).name ="pdm"
    set sheet = EXCEL.workbooks(1).sheets("pdm")

    ' 将一些文本放在工作表的第一行
    sheet.Cells(rowsNum, 1).Value = "表名"
    sheet.Cells(rowsNum, 2).Value = "表中文名"
    sheet.Cells(rowsNum, 3).Value = "表备注"
    sheet.Cells(rowsNum, 4).Value = "字段ID"
    sheet.Cells(rowsNum, 5).Value = "字段名"
    sheet.Cells(rowsNum, 6).Value = "字段中文名"
    sheet.Cells(rowsNum, 7).Value = "字段类型"
    sheet.Cells(rowsNum, 8).Value = "字段备注"
    sheet.cells(rowsNum, 9).Value = "主键"
    sheet.cells(rowsNum, 10).Value = "非空"
    sheet.cells(rowsNum, 11).Value = "默认值"
End Sub

'----------------------------------------------------------------------------------------------
' 导出目录结构
'----------------------------------------------------------------------------------------------
Sub ExportCatalog(tab)
    catalogNum = catalogNum + 1
    catalog.cells(catalogNum, 1).Value = tab.parent.name
    catalog.cells(catalogNum, 2).Value = tab.code
    catalog.cells(catalogNum, 3).Value = tab.comment
    '设置超链接
    catalog.Hyperlinks.Add catalog.cells(catalogNum,2), "",tab.code&"!A2"
End Sub 

'----------------------------------------------------------------------------------------------
' 导出sheet页
'----------------------------------------------------------------------------------------------
Sub ExportTable(tab, sheet)
    Dim col ' 运行列
    Dim colsNum
    colsNum = 0
    for each col in tab.columns
        colsNum = colsNum + 1
        rowsNum = rowsNum + 1
        sheet.Cells(rowsNum, 1).Value = tab.code
        'sheet.Cells(rowsNum, 2).Value = tab.name
        sheet.Cells(rowsNum, 2).Value = tab.comment
        'sheet.Cells(rowsNum, 4).Value = colsNum
        sheet.Cells(rowsNum, 3).Value = col.code
        'sheet.Cells(rowsNum, 4).Value = col.name
        sheet.Cells(rowsNum, 4).Value = col.datatype
        sheet.Cells(rowsNum, 5).Value = col.comment
        
        If col.Primary = true Then
            sheet.cells(rowsNum, 6) = "Y" 
        Else
            sheet.cells(rowsNum, 6) = "" 
        End If
        If col.Mandatory = true Then
            sheet.cells(rowsNum, 7) = "Y" 
        Else
            sheet.cells(rowsNum, 7) = "" 
        End If
        
        sheet.cells(rowsNum, 8).Value = col.defaultvalue
        '设置居中显示
        sheet.cells(rowsNum,6).HorizontalAlignment = 3
        sheet.cells(rowsNum,7).HorizontalAlignment = 3
    next
    output "Exported table: "+ +tab.Code+"("+tab.Name+")"
End Sub 

'----------------------------------------------------------------------------------------------
' 设置Excel目录页
'----------------------------------------------------------------------------------------------
Sub SetCatalog()
    Set EXCEL= CreateObject("Excel.Application")
    
    ' 使Excel通过应用程序对象可见。
    EXCEL.Visible = True
    EXCEL.workbooks.add(-4167)'添加工作表
    EXCEL.workbooks(1).sheets(1).name ="表结构"
    EXCEL.workbooks(1).sheets.add
    EXCEL.workbooks(1).sheets(1).name ="目录"
    set catalog = EXCEL.workbooks(1).sheets("目录")

    catalog.cells(catalogNum, 1) = "模块"
    catalog.cells(catalogNum, 2) = "表名"
    catalog.cells(catalogNum, 3) = "表注释"
    
    ' 设置列宽和自动换行
    catalog.Columns(1).ColumnWidth = 20
    catalog.Columns(2).ColumnWidth = 25
    catalog.Columns(3).ColumnWidth = 55
    
    '设置首行居中显示
    
    catalog.Range(catalog.cells(1,1),catalog.cells(1,3)).HorizontalAlignment = 3
    '设置首行字体加粗
    catalog.Range(catalog.cells(1,1),catalog.cells(1,3)).Font.Bold = True
End Sub 

'----------------------------------------------------------------------------------------------
' 新增sheet页
'----------------------------------------------------------------------------------------------
Sub AddSheet(sheetName)
    EXCEL.workbooks(1).Sheets(2).Select
    EXCEL.workbooks(1).sheets.add
    EXCEL.workbooks(1).sheets(2).name = sheetName
    set sheet = EXCEL.workbooks(1).sheets(sheetName)
    rowsNum = 1
    '将一些文本放在工作表的第一行
    sheet.Cells(rowsNum, 1).Value = "表名"
    'sheet.Cells(rowsNum, 2).Value = "表中文名"
    sheet.Cells(rowsNum, 2).Value = "表备注"
    'sheet.Cells(rowsNum, 4).Value = "字段ID"
    sheet.Cells(rowsNum, 3).Value = "字段名"
    'sheet.Cells(rowsNum, 4).Value = "字段中文名"
    sheet.Cells(rowsNum, 4).Value = "字段类型"
    sheet.Cells(rowsNum, 5).Value = "字段备注"
    sheet.cells(rowsNum, 6).Value = "主键"
    sheet.cells(rowsNum, 7).Value = "非空"
    sheet.cells(rowsNum, 8).Value = "默认值"
    
    '设置列宽
    sheet.Columns(1).ColumnWidth = 20
    sheet.Columns(2).ColumnWidth = 20
    sheet.Columns(3).ColumnWidth = 20
    sheet.Columns(4).ColumnWidth = 20
    sheet.Columns(5).ColumnWidth = 20
    sheet.Columns(6).ColumnWidth = 5
    sheet.Columns(7).ColumnWidth = 5
    sheet.Columns(8).ColumnWidth = 10

    '设置首行居中显示
    sheet.Range(sheet.cells(1,1),sheet.cells(1,8)).HorizontalAlignment = 3
    '设置首行字体加粗
    sheet.Range(sheet.cells(1,1),sheet.cells(1,8)).Font.Bold = True
    
    linkNum = linkNum + 1
    '设置超链接
    sheet.Hyperlinks.Add sheet.cells(1,1), "","目录"&"!B"&linkNum
End Sub 

```

<br>

# Excel批量导入到PDM

---

## 多sheet页导入

```vb
'******************************************************************************
'* Purpose:  从Excel中读取信息创建PDM模型
'* Title:
'* Category: 创建

'* Modified:
'* Use:      打开PDM，创建新的PDM，运行本脚本（Ctrl+Shift+X）
'*           Excel 格式要求
'*           MODEL Sheet
'*           |A     |B          |C          |D          |E      |F          |G          |H        |I    |J        |K      |
'*           主题域 |表注释 |表英文名称 |表中文名称 |列名   |列中文名称 |列注释 |数据类型 |主键 |是否为空 |默认值 |
'* Version:  1.0
'* Comment:
'******************************************************************************
 
Option Explicit
 
' Model sheet中的列信息
CONST CELL_A="A" '主题域(Pachage)
CONST CELL_B="B" '表注释
CONST CELL_C="C" '表英文名称
CONST CELL_D="D" '表中文名称
CONST CELL_E="E" '列名
CONST CELL_F="F" '列中文名称
CONST CELL_G="G" '列注释
CONST CELL_H="H" '数据类型
CONST CELL_I="I" '是否主键
CONST CELL_J="J" '是否可空
CONST CELL_K="K" '默认值
 
CONST str_iskey="Y"
'表的所属者
CONST str_username="srv"
CONST isclear_columns = true  '是否先删除表的所有列，如果是false则不会删除excel中没有的列，如果是true，则会重新创建相应表的所有列
 
' get the current active model
DIM mdl ' 定义当前的模型
SET mdl = ActiveModel '通过全局参数获得当前的模型
 
IF (mdl IS NOTHING) THEN
   MsgBox "没有选择模型，请选择一个模型并打开"
ELSEIF NOT mdl.IsKindOf(PdPDM.cls_Model) THEN
   MsgBox "当前选择的不是一个物理模型（PDM）."
ELSE
 
'选择需要导入的Excel文件
' 打开Excel
DIM xlApp   '定义Excel对象
SET xlApp  = CreateObject("Excel.Application")
xlApp.DisplayAlerts = FALSE
DIM xlBook  '定义Excel Sheet
SET xlBook = xlApp.WorkBooks.Open("F:\table.xlsx")
xlApp.Visible = TRUE
 
output "开始从Excel创建模型"
Create_From_Excel(xlBook)
output "模型创建完成，开始关闭Excel"
 
SET xlBook=NOTHING
xlApp.Quit
SET xlApp=NOTHING
 
END IF
 
PRIVATE SUB Create_From_Excel(xlBook)
  DIM xlsheet
  DIM rowcount
  dim pkg
 
  FOR EACH xlsheet IN xlBook.WORKSHEETS
	rowcount = xlsheet.UsedRange.Cells.Rows.Count
	output "本Excel["+xlsheet.name+"]共有行数为:"+CSTR(rowcount)
	IF rowcount>1 THEN
	  SET pkg = CreateOrReplacePackageByName( xlsheet.name , mdl)
	  Create_Model_From_Excel xlsheet,pkg 
	  SET xlsheet=NOTHING
	END IF
  NEXT
END SUB
 
'--------------------------------------------------------------------------------
'功能函数
'--------------------------------------------------------------------------------
PRIVATE SUB Create_Model_From_Excel(xlsheet,package)
	DIM Tab '定义数据表对象
	DIM col
	DIM tabcode
	DIM tabcode1
	DIM i
	DIM col_code
 
	FOR i=2 TO xlsheet.UsedRange.Cells.Rows.Count
		'判断是否需要创建新表对象
		tabcode1 = xlsheet.Range(CELL_C+CSTR(i)).Value
		IF tabcode1<>"" and tabcode<>tabcode1 THEN
			SET Tab=NOTHING 
			tabcode=tabcode1
			IF tabcode<>"" THEN
			    '判断表是否存在，如果不存在则创建，存在则直接返回表对象
				SET tab = CreateOrReplaceTableByCode(tabcode,package)
				'将表的所有列删除,如果需要重新创建表的列
				IF isclear_columns THEN
					DeleteTableColumns(tab)
				END IF
				'更新表的属性
				Tab.code=xlsheet.Range(CELL_C+CSTR(i)).Value
				Tab.name=xlsheet.Range(CELL_D+CSTR(i)).Value
				Tab.comment=xlsheet.Range(CELL_D+CSTR(i)).Value
				Tab.Description=xlsheet.Range(CELL_B+CSTR(i)).Value '注释
				'Tab.owner=FindUserByName(str_username)
				output "创建表模型OK:"+Tab.code+"——"+Tab.name
			END IF
		END IF
 
		IF NOT(Tab IS NOTHING) THEN '创建表的列
			col_code=xlsheet.Range(CELL_E+CSTR(i)).Value '列代码   	
			'判断是否已经存在列,不存在则创建
			SET col = CreateOrReplaceColumnByCode(col_code,Tab)
			'设置列属性
			col.code=xlsheet.Range(CELL_E+CSTR(i)).Value '列代码
			col.name=xlsheet.Range(CELL_F+CSTR(i)).Value '列名称
			col.comment=xlsheet.Range(CELL_F+CSTR(i)).Value '列注释
			col.Description=xlsheet.Range(CELL_G+CSTR(i)).Value '列注释
			col.DataType=xlsheet.Range(CELL_H+CSTR(i)).Value '列数据类型
			'列是否主键，如果是主键，则输出 Y
			IF CSTR(xlsheet.Range(CELL_I+CSTR(i)).Value)=str_iskey THEN
				col.primary= TRUE
			END IF
			output "更新表模型的列OK:"+Tab.code+"——"+col.code+"--"+col.name
		END IF
	NEXT
 
END SUB
 
'--------------------------------------------------------------------------------
'功能函数
'--------------------------------------------------------------------------------
PRIVATE FUNCTION CreateOrReplacePackageByName(name,model)
	DIM pkg 'Table 对象
	SET pkg = FindPackageByName(name,model)
	IF pkg IS NOTHING THEN
	  SET pkg = model.Packages.CreateNew()
	  pkg.SetNameAndCode name, name
	  pkg.PhysicalDiagrams.Item(0).SetNameAndCode name, name
	END IF
	SET CreateOrReplacePackageByName = pkg
END FUNCTION
 
PRIVATE FUNCTION CreateOrReplaceTableByCode(code,package)
	DIM tab 'Table 对象
	SET tab = FindTableByCode(code,package)
	IF tab IS NOTHING THEN
	  SET tab = package.Tables.CreateNew()
	  tab.SetNameAndCode code, code
	END IF
	SET CreateOrReplaceTableByCode = tab
END FUNCTION
 
PRIVATE FUNCTION CreateOrReplaceColumnByCode(code,table)
	DIM col 'Table 对象
	SET col =FindColumnByCode(code,table) 
	IF col IS NOTHING THEN
	  SET col =table.Columns.CreateNew
	  col.SetNameAndCode code , code
	END IF
	SET CreateOrReplaceColumnByCode = col
END FUNCTION
 
PRIVATE FUNCTION FindPackageByName(name,model)
	DIM pkg 'Table 对象
	SET FindPackageByName = NOTHING
	FOR EACH pkg IN model.Packages
		IF NOT pkg.isShortcut THEN
			IF pkg.name =name THEN
				SET FindPackageByName=pkg
				Exit FOR
			END IF
		END IF
	NEXT
	
END FUNCTION
 
PRIVATE FUNCTION FindTableByName(name,package)
	DIM Tab1 'Table 对象
	SET FindTableByName = NOTHING
	FOR EACH Tab1 IN package.Tables
		IF NOT Tab1.isShortcut THEN
			IF Tab1.name =name THEN
				SET FindTableByName=Tab1
				Exit FOR
			END IF
		END IF
	NEXT
END FUNCTION
 
PRIVATE FUNCTION FindTableByCode(code,package)
	DIM Tab1 'Table 对象
	SET FindTableByCode = NOTHING
	FOR EACH Tab1 IN package.Tables
		IF NOT Tab1.isShortcut THEN
			'OUTPUT "循环表:"+Tab1.name
			IF Tab1.code =code THEN
				SET FindTableByCode=Tab1
				Exit FOR
			END IF
		END IF
	NEXT
END FUNCTION
 
PRIVATE FUNCTION FindColumnByCode(code,tabobj)
	DIM col1 'Column 对象
	'OUTPUT "code:"+code
	SET FindColumnByCode = NOTHING
	FOR EACH col1 IN tabobj.Columns
		'OUTPUT "code2:"+col1.code
		IF col1.code =code THEN
			SET FindColumnByCode=col1
			EXIT FOR
		END IF
	NEXT
END FUNCTION
 
PRIVATE FUNCTION FindColumnByName(name,tabobj)
	DIM col1 'Column 对象
	'OUTPUT "codename:"+name
	SET FindColumnByName = NOTHING
	FOR EACH col1 IN tabobj.Columns
		IF col1.name =name THEN
			SET FindColumnByName=col1
			EXIT FOR
		END IF
	NEXT
END FUNCTION
 
PRIVATE FUNCTION FindDomainByName(dmname,mdl)
 
	DIM dm1 'Domain 对象
	SET FindDomainByName = NOTHING
 
	FOR EACH dm1 IN mdl.domains
		IF NOT dm1.isShortcut THEN
			IF dm1.name =dmname THEN
				SET FindDomainByName =dm1
				EXIT FOR
			END IF
		END IF
	NEXT
 
END FUNCTION
 
PRIVATE FUNCTION FindUserByName(username)
	DIM user1
	SET FindUserByName = NOTHING
	FOR EACH user1 IN mdl.users
		IF user1.name=username THEN
			SET FindUserByName=user1
			EXIT FOR
		END IF
	NEXT
 
END FUNCTION
 
' 删除表的所有列
PRIVATE SUB DeleteTableColumns(table)
  IF NOT table.isShortcut THEN  
   DIM col
   FOR EACH col IN table.columns  
  	'output "Column deleted :"+table.name
  	col.Delete
  	SET col = NOTHING
   NEXT
  END IF
END SUB

```

表格格式

| 主题域 | 表注释 | 表英文名称 | 表中文名称 | 列名 | 列中文名称 | 列注释 | 数据类型 | 主键 | 是否为空 | 默认值 |
| ------ | ------ | ---------- | ---------- | ---- | ---------- | ------ | -------- | ---- | -------- | ------ |
|        |        |            |            |      |            |        |          |      |          |        |

<br>

## 单sheet页导入

```vb
Option Explicit
 
Dim mdl ' the current model
Set mdl = ActiveModel
If (mdl Is Nothing) Then
   MsgBox "There is no Active Model"
End If
 
Dim HaveExcel
Dim RQ
RQ = vbYes 'MsgBox("Is Excel Installed on your machine ?", vbYesNo + vbInformation, "Confirmation")
If RQ = vbYes Then
   HaveExcel = True
   ' Open & Create Excel Document
   Dim x1  '
   Set x1 = CreateObject("Excel.Application")
   x1.Workbooks.Open "D:\testexcel2pdm1.xlsx"   '指定excel文档路径
   x1.Workbooks(1).Worksheets("Sheet1").Activate   '指定要打开的sheet名称
Else
   HaveExcel = False
End If
 
a x1, mdl
sub a(x1, mdl)
dim rwIndex   
dim tableName
dim colname
dim table
dim col
dim count
on error Resume Next
 
For rwIndex = 2 To 1000   '指定要遍历的Excel行标  由于第1行是表头，从第2行开始
        With x1.Workbooks(1).Worksheets("Sheet1")
            If .Cells(rwIndex, 1).Value = "" Then '如果遍历到第一列为空，则退出
               Exit For
            End If
            If .Cells(rwIndex, 3).Value = "" Then '如果遍历到第三列为空，则此行为表名
               set table = mdl.Tables.CreateNew     '创建表
                table.Name = .Cells(rwIndex , 1).Value '指定表名，第一列的值
                table.Code = .Cells(rwIndex , 2).Value 
                table.Comment = .Cells(rwIndex , 8).Value '指定表注释，第二列的值
                count = count + 1  
            Else
               set col = table.Columns.CreateNew   '创建一列/字段
               'MsgBox .Cells(rwIndex, 1).Value, vbOK + vbInformation, "列"            
                  col.Name = .Cells(rwIndex, 1).Value   '指定列名       
               'MsgBox col.Name, vbOK + vbInformation, "列"
               col.Code = .Cells(rwIndex, 2).Value   '指定列名                        
               col.DataType = .Cells(rwIndex, 5).Value '指定列数据类型           
                 'MsgBox col.DataType, vbOK + vbInformation, "列类型"               
               col.Comment = .Cells(rwIndex, 8).Value  '指定列说明
			   col.Precision = .Cells(rwIndex, 6).Value  '精度
			   If .Cells(rwIndex,7).Value="Y" Then
				col.Primary  = true  '是否主键
            End If
            End If      
        End With
Next
MsgBox "生成数据表结构共计 " + CStr(count), vbOK + vbInformation, "表"
 
Exit Sub
End sub

```

表格格式

| 表名/字段名 | 名称   | 类型    | 长度 | 数据类型   | 精度 | 是否主键 | 备注 |
| ----------- | ------ | ------- | ---- | ---------- | ---- | -------- | ---- |
| 表名1       | table1 |         |      |            |      |          |      |
| 字段1       | col1   | varchar | 3    | varchar(3) |      | Y        |      |
| 表名2       | table2 |         |      |            |      |          |      |
| 字段2       | col1   | varchar | 3    | varchar(3) |      | Y        |      |