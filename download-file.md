# download file 中文名乱码处理
	//tested browser: Chrom(56) Firefox(51) IE(11)
        response.setContentType("multipart/form-data");
	String userAgent = request.getHeader("User-Agent").toUpperCase()
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
