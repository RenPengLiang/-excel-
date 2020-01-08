##导入导出excel使用说明及范例：

本文档着重描述如何使用导入导出以及excel模板下载功能。目前支持导入导出的字典翻译、自定义导入导出字段、金额分元自动互转。

####导出数据到excel

------



###### 后端范例

* 首先使用代码生成器生成实体类xx.java，覆盖原entity类，保证该类上有注解@ExcelTemplateEntity(code = "xx", name = "xx"),code为实体类唯一编码，name为实体类唯一名称。
* 然后保证要导出的成员变量上标注了@ExcelTemplateField，注解ExcelTemplateField每项配置的详细说明见ExcelTemplateField.java文件代码注释说明。以下做简要描述：
  * sort - 数据排序值
  * code - 列 code
  * name - 列中文名
  * required - 是否必填,默认false
  * isAmount - 是否是金额,默认false
  * needImport - 是否作为导入字段，默认true
  * info - 列说明
  * 重要：当导出的数据中包含字典项时，则该字段必须定义为枚举类型，并实现IBaseEnum接口

* 示例代码

  ```java
  Controller层：
      
      // dto里面只有一个变量List<Long> ids，前台传递了则只导出指定记录，若未传递则默认导出200条（逻辑在service层自行判断）
      public void export(@RequestBody BatchExportDto dto) {
          xxService.export(dto);
      }
  
  Service层：
    接口定义：
      
    void export(BatchExportDto dto);
  
    接口实现：
          
    // 写在变量定义区域 XX.class:XX为前面生成的实体类名称
    private static final ExcelTemplateCore.DataStructureDefinition<TspMotorcade> 	OBJECT = parseType(XX.class).get();
    
    // 写在export方法实现区域 sourceList为查询结果
    final byte[] bytes = ExcelTemplateCore.exportData2003(OBJECT, sourceList);
    try {    
      HttpServletResponse httpServletResponse = ((ServletRequestAttributes) 	 RequestContextHolder.getRequestAttributes()).getResponse();    					  httpServletResponse.setContentType(MediaType.APPLICATION_OCTET_STREAM_VALUE);         httpServletResponse.setHeader("Content-Disposition", "attachment;filename=" + 		  URLEncoder.encode(OBJECT.getName() + ".xls", "UTF-8"));                  			  httpServletResponse.setHeader("Cache-Control", "No-cache");     			      		httpServletResponse.flushBuffer();    	    			 		 	  		 		  httpServletResponse.getOutputStream().write(bytes);} catch (Exception e) {        	  log.error("导出失败", e);
    }
  ```

#### 导入excel数据

------



######后端范例

* 首先使用代码生成器生成实体类xx.java，覆盖原entity类（若在做导出功能时，可忽略该步骤），保证该类上有注解@ExcelTemplateEntity(code = "xx", name = "xx"),code为实体类唯一编码，name为实体类唯一名称。

* 然后保证要导入的成员变量上标注了@ExcelTemplateField，注解ExcelTemplateField每项配置的详细说明见ExcelTemplateField.java文件代码注释说明。重要：**实体类中不需要导出的字段注解上@ExcelTemplateField 的配置项needImport置为false** 

* 示例代码

  ```java
  Controller层：
      
      // file为前台传递的文件
      public RestResponse<Object> batchImport(@RequestParam("file") MultipartFile file){         
          return xxService.batchImport(file);
      }
  
  Service层：
    接口定义：
      
    RestResponse<Object> batchImport(@RequestParam("file") MultipartFile file);
  
    接口实现：
  
    List<XX> sourceList = new ArrayList<>();
    final RestResponse<Object> response = new RestResponse<>();
    try {
      // 解析模板，解析失败则直接返回错误信息
      final ExcelTemplateResult<XX> xes = ExcelTemplateCore.resolveData2003(OBJECT, file.getInputStream());
      if (xes.getFailReasons() != null && xes.getFailReasons().size() > 0) {
          response.setMsg(xes.getFailReason());
          return response;
      }
      sourceList = xes.getItems();
      
      // TODO 对sourceList进一步处理
    }catch (IOException e) {
      e.printStackTrace();
    }
  ```



#### 下载excel模板

------



######后端范例

* 参考上述导入说明配置entity类

* 示例代码

  ```
  Controller层：
      
  	public void downloadTemplate() {    
  		xxService.downloadTemplate();
  	}
  
  Service层：
  
    接口定义：
      
    void downloadTemplate();
  	
    接口实现：
  
    public void downloadTemplate() {
      // 建立模板并输出
      final byte[] bytes = ExcelTemplateCore.generateTemplate2003(OBJECT);
      try {
          HttpServletResponse httpServletResponse = ((ServletRequestAttributes) RequestContextHolder.getRequestAttributes()).getResponse();							 								 	             		 	 	 	 		     httpServletResponse.setContentType(MediaType.APPLICATION_OCTET_STREAM_VALUE);
          httpServletResponse.setHeader("Content-Disposition", "attachment;filename=" + URLEncoder.encode(
                  OBJECT.getName() + ".xls", "utf-8"));
          httpServletResponse.setHeader("Cache-Control", "No-cache");
          httpServletResponse.flushBuffer();
          httpServletResponse.getOutputStream().write(bytes);
      } catch (Exception e) {
          log.error("下载失败" + e.getMessage());
          e.printStackTrace();
      }
    }
  	
  ```

###END

