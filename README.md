Introduction
============

`gsx-mockserver` is a mock version of the GSX SOAP API. It can be used for testing your GSX integration code or hosting demo environments, for example allowing you to test GSX-API functionality even if you don't have the necessary SSL certificates and firewall whitelistings for GSX access.

The mock responses are collected from example XML snippets in Apple's [API documentation][apidocs] and is therefore by no means exhaustive and 1:1 accurate, but it's definitely better than nothing and much faster to develop against than using the actual API test environments.


Installation
============

The server has only one external dependency - `lxml` which is used to parse the request XML. It should work with both Python 3 and 2, but is primarily developed and tested on 3.

	$ pip install lxml
	$ python serve.py
	Validating XML responses...
	GSX mock server serving on http://localhost:8080

You can also specify the port and address to serve on with the `-p` and `-a` arguments - check `-h` for the details.

Usage
=====

Simply point your GSX client code to the server you just started. The most traight-forward method would probably be to replace the API URL in your code.

If you're using [py-gsxws](https://github.com/filipp/py-gsxws), then you can simply point your GSX client to use the mock server by setting the `GSX_URL` environment variable to the URL of the mock server:

	GSX_URL=http://localhost:8080 python my_gsx_code_using_pygsxws.py


Variable reflection
===================

The mock server has a simple mechanism which can replace values in the response XML with values in the request allowing you to run tests that expect a certain value in the response. For example, if you're testing the "Mark Repair Complete" endpoint, it would be reasonable to expect the same repair dispatch ID to appear in the confirmation response that was sent in the request. To make this work, simply use a Python template placeholder inside the response XML file - naming it after the request parameter you want to use. For example, sending a `MarkRepairComplete` request such as this:

```xml
<MarkRepairCompleteRequest>
	<userSession>
		<userSessionId>Sdt7tXp2XytTEVwHBeDx6lHTXI3w9s+M</userSessionId>
    </userSession>
    <repairConfirmationNumbers>123456</repairConfirmationNumbers>
    <dispatchId>123456</dispatchId>
    </MarkRepairCompleteRequest>
</asp:MarkRepairComplete>
```

would result in the following response:

```xml
<MarkRepairCompleteResponse>
	<operationId>ykNJV0zy368v5uqnUYthI63zNSeip8/J</operationId>
		<repairConfirmationNumbers>
			<repairConfirmationNumber>123456</repairConfirmationNumber>
             <outcome>STOP/HOLD</outcome>
               <messages>Some Message1</messages>
               <messages>Some Message2</messages>
           </repairConfirmationNumbers>
           <repairConfirmationNumbers>
               <repairConfirmationNumber>123456</repairConfirmationNumber>
               <outcome>HOLD</outcome>
               <messages>Some Message1</messages>
               <messages>Some Message2</messages>
           </repairConfirmationNumbers>
           <repairConfirmationNumbers>
               <repairConfirmationNumber>123456</repairConfirmationNumber>
               <outcome>HOLD</outcome>
               <messages>Some Message1</messages>
               <messages>Some Message2</messages>
    </repairConfirmationNumbers>
</MarkRepairCompleteResponse>ionNumber>123456</repairConfirmationNumber>
```

Please note that due to inconsistencies in the response schema, this feature doesn't work with every request type.

Development
===========

If you run into an API call that the mock server cannot handle (resulting in a 404 response), just go to the [API docs][apidocs], find the example XML for the relevant call and copy/paste it into `responses/$SOAPAction.xml` where `$SOAPAction` equals the value of the `SOAPAction`header of the sent request (excluding the double quotes, for example Authenticate, MarkRepairComplete, UpdateCarryIn, etc)

[apidocs]: https://gsxapiut.apple.com/apidocs/ut/html/WSReference.html?user=asp

