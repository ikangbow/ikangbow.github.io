---
title: SpringSecurity验证流程解析
date: 2020-05-28 21:08:05
tags: docker
category: java
---

## SpringSecurity ##

1. TokenAuthenticationFilter
	if (httpRequest.getServletPath().equals(loginLink)) {
		//如果是登录或注销的话，设置不沿着过滤器向下
		doNotContinueWithRequestProcessing(httpRequest);
		checkLoginAnDoSomething(httpRequest, httpResponse, token);
	}
2. checkLoginAnDoSomething(httpRequest, httpResponse, token);

		example: Authorization=Basic YWRtaW46MTIzNDU2 (admin:123456)

		private boolean checkLoginAnDoSomething(HttpServletRequest httpRequest, HttpServletResponse httpResponse,
			String token) throws IOException {
			String authorization = httpRequest.getHeader(CacheConstant.AUTHORIZATION);
			ReturnStatus result = null;
			if (authorization != null) {
				result = checkBasicAuthorization(authorization, httpRequest, httpResponse, token);
			}
	
			if (null == result) {
				result = new ReturnStatus(false);
				result.getErrors().add(new MError(MErrorCode.e9000));
				result.setMessage(MErrorCode.e9000.desc());
			}
	
			String body;
			try {
				body = JsonUtils.toJson(result);// JsonParserFactory.getParser().toJson(result).toString();
				httpResponse.setContentType("text/html;charset=UTF-8");
				httpResponse.getWriter().write(body);
			} catch (Exception e) {
				throw new IOException(e);
			}
	
			return result.isSuccess();
	}

3. checkBasicAuthorization(authorization, httpRequest, httpResponse, token);

	    private ReturnStatus checkBasicAuthorization(String authorization, HttpServletRequest httpRequest,
				HttpServletResponse httpResponse, String token) throws IOException {
			StringTokenizer tokenizer = new StringTokenizer(authorization);
			if (tokenizer.countTokens() < 2) {
				return null;
			}
			if (!tokenizer.nextToken().equalsIgnoreCase(CacheConstant.BASIC_AUTH_PREFIX)) {
				return null;
			}
	
			String base64 = tokenizer.nextToken();
			String loginPassword = new String(Base64.decode(base64.getBytes(StandardCharsets.UTF_8)));
			tokenizer = new StringTokenizer(loginPassword, ":");
			String username = tokenizer.nextToken();
			String pwd = tokenizer.nextToken();
			String password = passwordEncoder.encodePassword(pwd, null);
			ReturnStatus status = checkUsernameAndPassword(username, password, httpRequest, httpResponse, token);
			// 登录成功后返回登录状态
			return status;
		}

3. private ReturnStatus checkUsernameAndPassword(String username, String password, HttpServletRequest httpRequest,HttpServletResponse httpResponse, String oldtoken)

		private ReturnStatus checkUsernameAndPassword(String username, String password, HttpServletRequest httpRequest,
				HttpServletResponse httpResponse, String oldtoken) throws IOException {
			ReturnStatus returnResult = new ReturnStatus(false);
			TokenInfo tokenInfo = authenticationService.authenticate(username, password, httpRequest);
			if (tokenInfo != null && tokenInfo.getUserDetails() != null) {
				VerifyContext verifyContext = (VerifyContext) tokenInfo.getUserDetails();
				Account account = (Account) verifyContext.getUser().getEntity();
				if (account.isLogin()) {
					if (null != oldtoken && !"".equals(oldtoken) && !"null".equals(oldtoken)
							&& !"undefined".equals(oldtoken)) {
						logger.info("tokenInfo.setToken use the oldtoken Token :{}", tokenInfo.getToken());
						tokenInfo.setToken(oldtoken);
					}
					logger.info("the new Token :{},entity:{}", tokenInfo.getToken(), tokenInfo);
					this.cacheManager.saveObject(CacheEnum.TOKEN, tokenInfo.getToken(), tokenInfo.getUserDetails(),
							CacheConstant.USER_SESSION_TIME);
					logger.info("the token:{}, save object:{}", tokenInfo.getToken(),
							this.cacheManager.getObject(CacheEnum.TOKEN, tokenInfo.getToken()));
					returnResult.setEntity(account);
					returnResult.setSuccess(true);
				} else {
					returnResult.getErrors().add(new MError(MErrorCode.e9001));
					returnResult.setMessage(MErrorCode.e9001.desc());
				}
				httpResponse.setHeader(CacheConstant.HEADER_TOKEN, tokenInfo.getToken());
			} else {
				logger.error("User {} ,Password {} Unauthorized!", username, password);
				returnResult.getErrors().add(new MError(MErrorCode.e9000));
				returnResult.setMessage(MErrorCode.e9000.desc());
			}
			return returnResult;
		}