## javascript-filedownload-authhorization
This smaple project is about downloading files from server with authorization token

I'm backend guy with old schools javascript knowledge. My UI team was facing file download issue, once we put wso2 API manager and authorization token before WSO2 ESB.

Spring boot microservice was able to show file download properly but when ever service was accessed via UI the download is always a corrupt file. 

I took this chanllange (even I have worked on js in year 2007 only ) done lot googling and but didn't find any working example for my project needs but multiple resources gave me good insight to understand the problem and to build possible solution.

I have added Html file with inline javascipt code.

Critical Points To Understand:
------------------------------
1. Need to read response data in String from bytes I have used toBinaryString function in inline js to achieve this.

2. Need to set dataURI 
          "data:application/octet-stream;base64,"+ btoa(data). 
Here btoa method creates a base-64 encoded ASCII string from a String object in which each character in the string is treated as a byte of binary data.

3. XMLHttpRequest object need to set Header for Authrozation with token like 
            xhr.setRequestHeader('Authorization', 'Bearer <your auth token>');

4. Override mimtype on XMLHttpRequest object with 
      xhr.overrideMimeType( "application/octet-stream; charset=x-user-defined;"); 

5. Completion of step 4 will make you able to download the file but it will not be able to retain the fileName transfered from  server. To achieve the file name retention I have used Kendo JS utility which provides SaveAs function with dataURI 		 defiend in step 2 and fileName. 
	
6. We need to retrieve the file name from Content-Disposition header from the response recevied from server. I found this 	header value is null in js code even fiddler shows me the value. I figured out the need to modify server side code as shown 	below in "Server Side Change" to make js to find this header value.

7. We need to use reex to find fileName from Content-Disposition header below is the technique I have used 
      var matchedFileNameArray = this.getResponseHeader("Content-Disposition").match(/filename=['"](.*)['"]$/);

8. Now in Kendo.SaveAs use dataURI and fileName as below
      kendo.saveAs({
							dataURI: data,
							fileName: matchedFileNameArray[1]
						});
		}, false);

9. Now you should be able to find file download happens correctly with the same fileName delivered by server.

Server Side Change:
----------------------
Service from server should set AccessControlExposeHeaders in response headers. Below one is the way I have added in my spring boot service. 

        List<String> acexposeHeader = new ArrayList<String>();
        acexposeHeader.add("Content-Description");
        acexposeHeader.add("Content-Disposition");
        acexposeHeader.add("Content-Type");
        respHeaders.setAccessControlExposeHeaders(acexposeHeader);

Hope this helps. I'm aware this is old school jaavscript style code which can be more concise with angular js code , I'm learning now :)

You can pull/download the code and make changes for GET URL and auth token and have server side changes in place then functionality will work for you.
