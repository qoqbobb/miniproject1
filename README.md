# portfolio - 1차 팀 미니 프로젝트(DA !t DA Market)




# 개발목적

- 처음하는 팀 프로젝트로서 SpringMVC의 MVC숙달
- 처음하는 팀 프로젝트로서 깃 협업&숙달

------------------------------------------------------------------------------------------------------------------------------------------

# 개발환경
>front-end
- bootstrap 
- jquery 

>back-end
- Java
- spring
- lombok 
- oracle DB
- mybatis 
- tomcat
- maven 





------------------------------------------------------------------------------------------------------------------------------------------

# DB 모델링
<div>
  <img src="https://user-images.githubusercontent.com/105841315/190603755-18082878-bd22-4112-90e4-53ec29d8af2b.png">
</div>



------------------------------------------------------------------------------------------------------------------------------------------

# 기능별 주요 로직
>  상품등록 이미지 업로드&저장 
- 상품 등록<br>
### register.jsp: 등록 페이지.
~~~jsp
<form action="/board/register" method="post">
   <input type="hidden" name="${_csrf.parameterName}" value="${_csrf.token}"/>  
    <div class="form-group">
      <label for="title">상품명</label>
      <input type="text" class="form-control" id="title"name="title"
       maxlength="100" required="required"
    </div>
    <div class="form-group">
   <label for="content">상품 정보</label>
   <textarea class="form-control" rows="5" id="content"
    name="pinfo" placeholder="내용 작성"></textarea>
 </div>
    <div class="form-group">
      <label for="writer">판매자</label>
      <input class="form-control" name='celler' value='<sec:authentication property="principal.username"/>' readonly="readonly" >
    </div>
    <div class="form-group">
      <label for="writer">가격</label>
      <input type="text" class="form-control" id="writer"
       placeholder="가격" name="price">
    </div>
    <input type="file"  id ="fileItem" name='uploadFile' style="height: 30px;">
  <div id="uploadResult">
</div>
<hr>
    <button type="submit" class="btn btn-default">등록</button>
</div>
</form>
~~~

### BoardController : 등록 로직 실행 후 상품리스트로 이동
 ~~~java
 @PreAuthorize("isAuthenticated()")
	@PostMapping("/register")
	public String register(BoardVO vo, RedirectAttributes rttr) {
		
		service.register(vo);
		log.info("board 등록 : "+vo);
		return "redirect:/board/list";
	}
 ~~~
 
### 상품등록
<div>
  <img src="https://user-images.githubusercontent.com/105841315/191143007-c847b3ed-c9de-40b9-88dd-9d8ca3958166.gif">
</div>




- 이미지 업로드&저장
### register.jsp 중 파일 업로드를 처리하는 ajax
~~~JavaScript
$("input[type='file']").on("change", function(e){
	if($(".imgDeleteBtn").length > 0){
		deleteFile();
	}
	let formData = new FormData();
	let fileInput = $('input[name="uploadFile"]');
	let fileList = fileInput[0].files;
	let fileObj = fileList[0];

	 if(!fileCheck(fileObj.name, fileObj.size)){
		return false;
	} 
	
		formData.append("uploadFile", fileObj);
	
	$.ajax({
		url: '/uploadAjaxAction',
    	processData : false,
    	contentType : false,
    	data : formData,
    	
    	 beforeSend: function(xhr) {
    			xhr.setRequestHeader(csrfHeaderName,csrfTokenValue);
    		},
    	
    	type : 'POST',
    	dataType : 'json',
    	success : function(result){
	    		console.log(result);
	    		showUploadImage(result);
	    	},
			error : function(result){
				alert("이미지 파일이 아닙니다.");
	    	}
	});
});
~~~
### : 글을 등록할때 파일 정보를 참조할 수 있도록 hidden으로 함께 전송
~~~JavaScript
if(!uploadResultArr || uploadResultArr.length == 0){return}
	
	let uploadResult = $("#uploadResult");
	
	let obj = uploadResultArr[0];
	
	let str = "";
	
	let fileCallPath = encodeURIComponent(obj.uploadPath.replace(/\\/g, '/') + "/s_" + obj.uuid + "_" + obj.fileName);
	
	str += "<div id='result_card'>";
	str += "<img src='/display?fileName=" + fileCallPath +"'>";
	str += "<div class='imgDeleteBtn' data-file='"+fileCallPath+"'>x</div>";
	str += "<input type='hidden' name='imageList[0].fileName' value='"+ obj.fileName +"'>";
	str += "<input type='hidden' name='imageList[0].uuid' value='"+ obj.uuid +"'>";
	str += "<input type='hidden' name='imageList[0].uploadPath' value='"+ obj.uploadPath +"'>";
	str += "</div>";		
	
		uploadResult.append(str);  
~~~

### UploadController.java : 파일 등록 부분
~~~java
@PostMapping(value="/uploadAjaxAction", produces = MediaType.APPLICATION_JSON_UTF8_VALUE)
	 public ResponseEntity<List<AttachImageVO>> uploadAjaxActionPOST(MultipartFile[] uploadFile){
		
		String uploadFolder = "C:\\upload";
		SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd");
		Date date = new Date();
		String str = sdf.format(date);
		String datePath = str.replace("-", File.separator);
		File uploadPath = new File(uploadFolder, datePath);
		
		if(uploadPath.exists() == false) {
			uploadPath.mkdirs();
		}
		List<AttachImageVO> list = new ArrayList();
	   for(MultipartFile multipartFile : uploadFile) {
		   AttachImageVO vo = new AttachImageVO();
			String uploadFileName = multipartFile.getOriginalFilename();
			vo.setFileName(uploadFileName);
			vo.setUploadPath(datePath);
			String uuid = UUID.randomUUID().toString();
			vo.setUuid(uuid);
			uploadFileName = uuid + "_" + uploadFileName;
			File saveFile = new File(uploadPath, uploadFileName);
			try {
				multipartFile.transferTo(saveFile);
				File thumbnailFile = new File(uploadPath, "s_" + uploadFileName);	
				BufferedImage bo_image = ImageIO.read(saveFile);
					
					double ratio = 3;
					int width = (int) (bo_image.getWidth() / ratio);
					int height = (int) (bo_image.getHeight() / ratio);					
				Thumbnails.of(saveFile)
		        .size(width, height)
		        .toFile(thumbnailFile);
			} catch (Exception e) {
				e.printStackTrace();
			} 
			list.add(vo);
		    } 
	   ResponseEntity<List<AttachImageVO>> result = new ResponseEntity<List<AttachImageVO>>(list, HttpStatus.OK);
		
		return result;
	   }
~~~
### 검색
<div>
 <img src="https://user-images.githubusercontent.com/19407579/69799597-ed6ac780-1216-11ea-9af9-b569be35792c.gif">
</div>
<br>

>이미지 게시판 구현 

### BoardServiceImpl : 상품이 가지고 있는 이미지 가져오는 로직

~~~java
@Override
	public List<BoardVO> showlist() {
		List<BoardVO> list = mapper.getlist();
		list.forEach(pd ->{
			Long pdId = pd.getPno();
			List<AttachImageVO> imageList = mapper.getAttachList(pdId);
			pd.setImageList(imageList);
		});
		return list;
	}
~~~
### list.jsp DB에 저장되어있는 이미지 주소 불러오기
~~~jsp
    <div class="card border-0 transform-on-hover">
	                	<a class="lightbox" href='/board/get?pno=<c:out value="${list.pno}"/>
	                		<div class="image_wrap" data-pno="${list.imageList[0].pno}" data-path="${list.imageList[0].uploadPath}" 
	                					data-uuid="${list.imageList[0].uuid}" data-filename="${list.imageList[0].fileName}">
						<img alt="Card Image" class="card-img-top">
					</div>
~~~
### 파일 삭제 결과
<div>
 <img src="https://user-images.githubusercontent.com/19407579/69810742-ea300580-122f-11ea-8198-fc578566bc40.gif">
</div>

------------------------------------------------------------------------------------------------------------------------------------------



# 스프링으로 진행하며 느낀점
- jsp model2 mvc에 비해 의존성 주입을 활용한 어노테이션으로 객체 자동 생성이 편하다.
- jsp model2 mvc에서 요청한 URL을 잘라서 String 변수command 를 통해  요청을 해결하는 것보다 @requestmapping 혹은
@postmapping, @getmapping 한방으로 끝나는 요청 분기 나누는게 편하다.
- jsp model2 mvc의 preparedStatment 에 비해 월등히 편한 mybatis
- junit의 단위테스트의 편리함. 꼭 실행화면에서 안해도 된다. was를 사용하지 않아도 되서 실행시간을 엄청 절감 할 수 있다.
일반 main메소드에서 테스트를 한다고하면 해당 클래스를 만들고 메소드를 호출해야하는 번거로움 있다.
그냥 junit 테스트코드에서 단위테스트하면 여러 메소드가 바로 이루어지고 junit탭에 정리가 되어서 나와 에러가 뜨면 바로바로 잡을수 있어서 좋았음.
기능별로 하나의 단위테스트 모듈을 만들어 두면 두고두고 써먹을 수 있을 것 같다.
- 권한에 따른 흐름제어를 내가 1부터 코딩 하는것이 아니고 spring security를 통해 쉽게 제어할 수 있다.
- lombok을 통해 setter, getter, tostring 작성을 생략 가능해 편하다.



