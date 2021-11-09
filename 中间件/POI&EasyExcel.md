[toc]

# Apache-POI

## 基本功能

### 结构:

- **HSSF---提供读写Microsoft Excel格式档案的功能**
- **XSSF---提供读写Microsoft Excel OOXML格式档案的功能**
- HWPF---提供读写Microsoft Word格式档案的功能
- HSLF---提供读写Microsoft PowerPoint格式档案的功能
- HDGF---提供读写Microsoft Visio格式档案的功能



### **文件解压文件读取通过文件形式**

![poi1](http://qiliu.luxiaobai.cn/img/poi1.png)



### 避免将全部数据一次加载到内存

采用sax模式一行一行解析，并将一行的解析结果以观察者的模式通知处理。

![poi2](http://qiliu.luxiaobai.cn/img/poi2.png)



## POI-Excel写

```xml
<!--xls(03)-->
<dependency>
    <groupId>org.apache.poi</groupId>
    <artifactId>poi</artifactId>
    <version>4.1.0</version>
</dependency>

<!--xlsx(07)-->
<dependency>
    <groupId>org.apache.poi</groupId>
    <artifactId>poi-ooxml</artifactId>
    <version>3.9</version>
</dependency>

<!--日期格式化工具-->
<dependency>
    <groupId>joda-time</groupId>
    <artifactId>joda-time</artifactId>
    <version>2.10.1</version>
</dependency>

<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.13.2</version>
    <scope>test</scope>
</dependency>
```



```java
public static void testWrite07() throws IOException {
		// 1 创建一个工作傅
		Workbook workbook = new XSSFWorkbook();
           // Workbook workbook = new HSSFWorkbook();
		// 2 创建一个工作表
		Sheet sheet = workbook.createSheet("路小白观众统计表");
		// 3 创建一个行
		Row row1 = sheet.createRow(0);
		// 4 创建一个单元格
		// (1,1)
		Cell cell11 = row1.createCell(0);
		cell11.setCellValue("今日新增观众");
		// (1,2)
		Cell cell12 = row1.createCell(1);
		cell12.setCellValue(666);

		Row row2 = sheet.createRow(1);
		Cell coll21 = row2.createCell(0);
		coll21.setCellValue("统计时间:");
		Cell coll22 = row2.createCell(1);
		String time = new DateTime().toString("yyyy-MM-dd HH:mm:ss");
		coll22.setCellValue(time);

		// 生成一张表 07版本使用xlsx结尾\03版本使用xls结尾
		FileOutputStream fileOutputStream = new FileOutputStream(PATH + "路小白07.xlsx");
		workbook.write(fileOutputStream);
		//关闭流
		fileOutputStream.close();
		System.out.println("Excel生成完毕");
	}
```



## 03版本VS07版本

大文件写HSSF



### 03版本

缺点:最多只能处理65535行,否则会抛出异常

```java
Exception in thread "main" java.lang.IllegalArgumentException: Invalid row number (65536) outside allowable range (0..65535)
```

优点:过程中写入缓存,不操作磁盘,最后一次性写入磁盘,速度快

```java
public static void testWrite03BigDate() {
		// 时间
		long start = System.currentTimeMillis();

		Workbook workbook = new HSSFWorkbook();
		Sheet sheet = workbook.createSheet();
		for (int i = 0; i < 65536; i++) {
			Row row = sheet.createRow(i);
			for (int j = 0; j < 20; j++) {
				Cell cell = row.createCell(j);
				cell.setCellValue(j);
			}
		}
		System.out.println("over");
		try {
			FileOutputStream fileOutputStream= new FileOutputStream(PATH + "路小白03.xls");			
			workbook.write(fileOutputStream);
			fileOutputStream.close();
		} catch (Exception e) {
			// TODO: handle exception
			e.printStackTrace();
		}

		long end = System.currentTimeMillis() - start;
		System.out.println(">>>" + end);
	}
```



大文件写XSSF

### 07版本

缺点:写数据时速度快非常慢,非常耗内存,也会发生内存溢出,如100万条

优点:**可以写较大的数据量,如20万条** 

```java
public static void testWrite03BigDate() {
		// 时间
		long start = System.currentTimeMillis();

		Workbook workbook = new XSSFWorkbook();
		Sheet sheet = workbook.createSheet();
		for (int i = 0; i < 35537; i++) {
			Row row = sheet.createRow(i);
			for (int j = 0; j < 20; j++) {
				Cell cell = row.createCell(j);
				cell.setCellValue(j);
			}
		}
		System.out.println("over");
		try {
			FileOutputStream fileOutputStream= new FileOutputStream(PATH + "路小白03.xlsx");			
			workbook.write(fileOutputStream);
			fileOutputStream.close();
		} catch (Exception e) {
			// TODO: handle exception
			e.printStackTrace();
		}

		long end = System.currentTimeMillis() - start;
		System.out.println(">>>" + end);
	}
```



大文件写SXSSF

优点:可以写非常大的数据量,如100万条甚至更多条,写数据速度快,占用更少的内存

注意:

- 过程中会产生临时文件,需要清理临时文件
- 默认由100条记录杯保存在内存中,如果超过这个数量,则最前面的数据被写入临时文件
- 如果想自定义内存中数据的数量,可以使用new SXSSFWorkbook(数量)

SXSSFWorkbook来至官方的解释:实现“BigGridDem”策略的流式XSS Workbook版本.这允许写入非常大的文件而不会耗尽内存,因为任何时候只有可配置的行部分被保存在内存中. 

```java
public static void testWrite03BigDate() {
		// 时间
		long start = System.currentTimeMillis();

		Workbook workbook = new SXSSFWorkbook();
		Sheet sheet = workbook.createSheet();
		for (int i = 0; i < 65537; i++) {
			Row row = sheet.createRow(i);
			for (int j = 0; j < 20; j++) {
				Cell cell = row.createCell(j);
				cell.setCellValue(j);
			}
		}
		System.out.println("over");
		try {
			FileOutputStream fileOutputStream= new FileOutputStream(PATH + "路小白07.xlsx");			
			workbook.write(fileOutputStream);
			fileOutputStream.close();
			//清除临时文件
			((SXSSFWorkbook) workbook).dispose();
		} catch (Exception e) {
			// TODO: handle exception
			e.printStackTrace();
		}

		long end = System.currentTimeMillis() - start;
		System.out.println(">>>" + end);
	}
```



## POI-Excel读

```java
 public static void testRead() throws Exception {
		FileInputStream fileInputStream = new FileInputStream(PATH + "路小白.xls");
		Workbook workbook = new HSSFWorkbook(fileInputStream);
		Sheet sheet = workbook.getSheetAt(0);
		Row row = sheet.getRow(0);
		Cell cell = row.getCell(1);
		System.out.println(cell.getNumericCellValue());
		fileInputStream.close();

	}
```



注意获取值的类型

### **读取不同的类型**

```java
// 读取不同的数据类型
	public void testReactCellType() throws Exception {
		// 获取文件流
		FileInputStream fileInputStream = new FileInputStream(PATH + "路小白.xls");
		Workbook workbook = new HSSFWorkbook(fileInputStream);
		Sheet sheet = workbook.getSheetAt(0);
		// 获取标题内容
		Row rowtitle = sheet.getRow(0);
		if (rowtitle != null) {
			int cellCount = rowtitle.getPhysicalNumberOfCells();
			for (int cellNum = 0; cellNum < cellCount; cellNum++) {
				Cell cell = rowtitle.getCell(cellNum);
				if (cell != null) {
					String cellValue = cell.getStringCellValue();
					System.out.println(cellValue + " | ");
				}

			}
			System.out.println();
		}

		// 获取表中内容
		int rowCount = sheet.getPhysicalNumberOfRows();
		for (int rowNum = 1; rowNum < rowCount; rowNum++) {
			Row row = sheet.getRow(rowNum);
			if (row != null) {
				// 读取列
				int cellCount = row.getPhysicalNumberOfCells();
				for (int cellNum = 0; cellNum < cellCount; cellNum++) {
					System.out.println("[" + (rowNum + 1) + "-" + (cellNum + 1) + "]");
					Cell cell = row.getCell(cellNum);
					// 匹配列的数据类型
					if (cell != null) {
						int cellType = cell.getCellType();
						String cellValue = "";
						switch (cellType) {
						case HSSFCell.CELL_TYPE_STRING: // 字符串
							System.out.println("[String]");
							cellValue = cell.getStringCellValue();
							break;
						case HSSFCell.CELL_TYPE_BOOLEAN:// 布尔
							System.out.println("[BOOLEAN");
							cellValue = String.valueOf(cell.getBooleanCellValue());
							break;
						case HSSFCell.CELL_TYPE_BLANK:// 空
							System.out.println("[BANK]");
							break;
						case HSSFCell.CELL_TYPE_NUMERIC:// 数字(日期、普通数字)
							System.out.println("[NUMRIC]");
							if (HSSFDateUtil.isCellDateFormatted(cell)) {// 日期
								System.out.println("日期");
								Date date = cell.getDateCellValue();
								cellValue = new DateTime(date).toString();
							} else {
								// 不是日期格式,防止数字过长
								System.out.println("[转换为字符串输出]");
								cell.setCellType(HSSFCell.CELL_TYPE_STRING);
								cellValue = cell.toString();
							}
							break;
						}
						System.out.println(cellValue);
					}
				}
			}

		}
		fileInputStream.close();
	}
```



### **计算公式**

```java
public void testFormula(FileInputStream fileInputStream) throws Exception{
		Workbook workbook = new HSSFWorkbook(fileInputStream);
		Sheet sheet = workbook.getSheetAt(0);
		Row row = sheet.getRow(4);
		Cell cell = row.getCell(0);
		
		//拿到计算公式 eval
		FormulaEvaluator formulaEvaluator = new HSSFFormulaEvaluator((HSSFWorkbook) workbook);
		
		//输出单元格内容
		int cellType = cell.getCellType();
		switch (cellType) {
		case Cell.CELL_TYPE_FORMULA://公式
			String formula = cell.getCellFormula();
			System.out.println(formula);
			//计算
			CellValue evaluate = formulaEvaluator.evaluate(cell);
			String cellValue = evaluate.formatAsString();
			System.out.println(cellValue);
			break;

		default:
			break;
		}
	}

```





# alibaba-easyExcel

https://github.com/alibaba/easyexcel

https://www.yuque.com/easyexcel/doc/about

EasyExcel是阿里巴巴开源的一个excel处理框架,以使用简单、节省内存著称

Easy Excel能大大减少占用内存的主要原因是在解析Excel时没有将文件数据一次性全部加载到内存中,而是从磁盘上一行行读取数据,逐个解析

![easy1](http://qiliu.luxiaobai.cn/img/easy1.png)

```java
public static List<DemoData> init() {
		List<DemoData> list = new ArrayList<DemoData>();
		for (int i = 0; i < 10; i++) {
			DemoData data = new DemoData();
			data.setString("字符串" + i);
			data.setDate(new Date());
			data.setDoubleData(0.56);
			list.add(data);
		}
		return list;
	}

	public static void simpleWrite() {
		// 写法1
		String fileName = PATH + System.currentTimeMillis() + ".xlsx";
		// 这里 需要指定写用哪个class去写，然后写到第一个sheet，名字为模板 然后文件流会自动关闭
		// 如果这里想使用03 则 传入excelType参数即可
		EasyExcel.write(fileName, DemoData.class).sheet("模板").doWrite(init());

		// 写法2
		fileName = PATH + System.currentTimeMillis() + ".xlsx";
		// 这里 需要指定写用哪个class去写
		ExcelWriter excelWriter = null;
		try {
			excelWriter = EasyExcel.write(fileName, DemoData.class).build();
			WriteSheet writeSheet = EasyExcel.writerSheet("模板").build();
			excelWriter.write(init(), writeSheet);
		} finally {
			// 千万别忘记finish 会帮忙关闭流
			if (excelWriter != null) {
				excelWriter.finish();
			}
		}
	}
```

