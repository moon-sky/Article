# 图灵iOS SDK开发需求 #
## 语音识别 ##

    封装百度语音识别SDK，包含以下类  
1. 资料地址[http://yuyin.baidu.com/asr/download](http://yuyin.baidu.com/asr/download)
1. RecognizeManager implements VoiceClientStatusChangeListener **class**
	1. startRecognize()
	2. stopRecognize()
	3. setRecognizeListener(RecognizeListener)

1. RecognizeListener **interface**
	1. public void onRecognizeResult(String result);
	2. public void onRecognizeError(String error);
## 语音朗读 ##
    封装百度语音合成SDK


1. 资料地址[http://yuyin.baidu.com/tts/download](http://yuyin.baidu.com/tts/download "tts")
2. TTSManger **class**包含以下方法
	
	 		/**开始tts合成
	    	 * @param ttsContent 待合成的文本内容
	    	 * @return 0 成功 其他都是调起失败
	    	 */
	    	public int startTTS(String ttsContent){}
	 
	    	/**
	    	 * 取消tts合成
	    	 */
			public void cancelTTS(){}	
	      
	    	/**
	    	 * 暂停tts合成
	    	 */
	    	public void pauseTTS(){	}

		    /**
	    	 * 恢复tts合成
	    	 */
	    	public void resumeTTS(){	}
			/**
			 * 设置百度tts相关参数
			 */
			private void setParams() {
		        speechSynthesizer.setParam(SpeechSynthesizer.PARAM_SPEAKER, "0");
		        speechSynthesizer.setParam(SpeechSynthesizer.PARAM_VOLUME, "5");
		        speechSynthesizer.setParam(SpeechSynthesizer.PARAM_SPEED, "5");
		        speechSynthesizer.setParam(SpeechSynthesizer.PARAM_PITCH, "5");
		        speechSynthesizer.setParam(SpeechSynthesizer.PARAM_AUDIO_ENCODE, SpeechSynthesizer.AUDIO_ENCODE_AMR);
		        speechSynthesizer.setParam(SpeechSynthesizer.PARAM_AUDIO_RATE, SpeechSynthesizer.AUDIO_BITRATE_AMR_15K85);
		
		    }
## 语义识别 ##
1. get请求图灵api
2. 方法：http://www.tuling123.com/openapi/api?key=用户的apikey&info=用户的命令&userid=用户的自己的id**这个后面会提到**
3. TuringRequestManager
	
	1. getUserID() 方法同样是通过http 请求http://www.tuling123.com/openapi/getuserid.do?key=用户的设备id
	1. startTuringRequest(String userid,String input)
	2. setRequestListenr(TuringRequestListener listener)

1. TuringRequestListener
	1. onRequestSucess(String result)
	2. onRequestError(String error)

## 上传通讯录 ##
1. 使用post方式上传	
    
	    JSONObject jsonObject=new JSONObject();
	    jsonObject.put("actiontype", "toy_assist_userid");
	    jsonObject.put("userid", config.getTuring_userid());
	    jsonObject.put("addressbook", localAddressList);
	    String pamars=jsonObject.toString();
1. UpLoadContactManager
	1. startUpload()
	2. setUploadListenr(UploadContactListener listener)
	
1. UploadContactListener 
	1. uploadSuccess()
	2. uploadFail