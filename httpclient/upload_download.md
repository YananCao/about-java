# use httpclient
## file upload
		public static ResourceFileData getUploadResponse(MultipartFile file, String 				uploadUrl){
			String result = "";
			CloseableHttpResponse httpResponse=null;
			CloseableHttpClient httpClient=HttpClients.createDefault();
			HttpPost  hp = new HttpPost(uploadUrl);
			MultipartEntityBuilder builder = MultipartEntityBuilder.create()
					.setMode(HttpMultipartMode.BROWSER_COMPATIBLE);
			try {
				InputStreamBody stream = new InputStreamBody(file.getInputStream(),
						file.getOriginalFilename());
				builder.addPart("file", stream).setCharset(CharsetUtils.get("UTF-8"));
				HttpEntity multipart = builder.build();
				hp.setEntity(multipart);
				httpResponse = httpClient.execute(hp);
				HttpEntity entity = httpResponse.getEntity();
				if(null != entity){
					result = EntityUtils.toString(entity);
				}
			} catch (IOException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}finally{
				try {
					httpResponse.close();
					httpClient.close();
				} catch (IOException e) {
					e.printStackTrace();
				}
			}
			if(StringUtils.isEmpty(result)){
				return new ResourceFileData();
			}
			ResourceFileData res = JsonUtil.fromJson(result,
					ResourceFileData.class);
			return res;
		}


## file download
		public static boolean getPdfFile(
				String url, String fileName, String fileUrl,
				String clearFileUrl, String userAgent,
				HttpServletResponse response){
			CloseableHttpClient httpClient = HttpClients.createDefault();
			HttpPost httpPost = new HttpPost(url);
			httpPost.addHeader("Content-type","application/json");
			StringEntity se=new StringEntity(fileUrl,"UTF-8");
			httpPost.setEntity(se);
			CloseableHttpResponse httpResponse = null;
			ServletOutputStream sos = null;
			try {
				httpResponse = httpClient.execute(httpPost);
				if(httpResponse.getStatusLine().getStatusCode() == 200){
					Header[] fileDesc = httpResponse.getHeaders("Content-Disposition");
				Map<String, String> params = new HashMap<>();
				params.put("file",
						fileDesc[0].getValue().split("=")[1]);
				getResonse(clearFileUrl, JsonUtil.toJson(params));
				HttpEntity entity = httpResponse.getEntity();
				InputStream is = entity.getContent();
				response.setContentType("multipart/form-data");
				if(userAgent.indexOf("FIREFOX") > 0){ //firefox模式
					fileName = new String(fileName.getBytes("UTF-8"), "ISO-8859-1");
				}else{  //非firefox模式
					fileName = URLEncoder.encode(fileName, "UTF-8");
				}
				response.setHeader("Content-Disposition", "attachment;fileName="+
				fileName);
				byte[] tmp = new byte[1024];
				sos = response.getOutputStream();
				int len = -1;
				while((len=is.read(tmp)) > -1){
				sos.write(tmp, 0, len);
				}
					return true;
				}
			} catch (IOException e) {
				e.printStackTrace();
			} finally {
				if(sos != null){
					try {
						sos.flush();
						sos.close();
					} catch (IOException e) {
						e.printStackTrace();
					}
				}
				try {
					if(httpResponse != null)
						httpResponse.close();
					httpClient.close();
				} catch (IOException e) {
					e.printStackTrace();
				}
			}
			return false;
		}
