ReCaptcha on Django

1. Get your public and private recaptcha keys
2. Add keys to your settings.py: 
        RECAPTCHA_PUBLIC_KEY = 'insert your public key'
        RECAPTCHA_PRIVATE_KEY = 'insert your private key'
3. Download Python's Recaptcha-client
        1. cd to the directory, e.g. recaptcha-client-x.x.x
        2. python setup.py install 
                                                  (or)
       by using terminal,
            sudo pip install recaptcha-client
4. def displayhtml(public_key,  attrs,  use_ssl=False,  error=None)
   def submit(recaptcha_challenge_field,   recaptcha_response_field,  private_key,   remoteip, use_ssl=False)

5.  views.py

from recaptcha.client import captcha
from django.conf import settings
from django.shortcuts import render_to_response
import json
from django.http.response import HttpResponse


def index(request):
    c = captcha.displayhtml(public_key=settings.RECAPTCHA_PUBLIC_KEY,use_ssl = True)
    return render_to_response('reg.html',{'captcha':c})


def NewReg(request):
    resp = captcha.submit(recaptcha_challenge_field=request.POST.get('recaptcha_challenge_field'), \
                              recaptcha_response_field=request.POST.get('recaptcha_response_field'), \
                              private_key=settings.RECAPTCHA_PRIVATE_KEY, \
                              remoteip=request.META['REMOTE_ADDR'])
    
    if not resp.is_valid:
        data= {'error':True, 'Message': 'security text mismatch !!!'}
        return HttpResponse(json.dumps(data))

6. application.html

<div id="div-reg"> 
	<form id="reg">
		<legend>New User</legend>
		<div>
			<label>Name</label>
                        <input type="text" placeholder="profile" name="name">
		        <label>Password</label>
			<input type="password" name="pwd">
			<div id="captcha">{{captcha|safe}}</div>
		</div>
		<button type="submit">Save</button>
 	 </form>
</div>

<script type="text/javascript">
    $(document).ready(function(){
        $("form#reg").submit(function(e){
	      e.preventDefault();
	      $.post('/reg/newreg/',$('form#reg').serialize(), function(data){
			console.log(data);
	      },'json');	
	});
    });
</script>
7.

8.

9.


10.

from recaptcha.client import captcha
from django.conf import settings
from django.shortcuts import render_to_response
import json
from django.http.response import HttpResponse


def index(request):
    c = captcha.displayhtml(public_key=settings.RECAPTCHA_PUBLIC_KEY,use_ssl = True)
    return render_to_response('reg.html',{'captcha':c})


def NewReg(request):
    resp = captcha.submit(recaptcha_challenge_field=request.POST.get('recaptcha_challenge_field'), \
                              recaptcha_response_field=request.POST.get('recaptcha_response_field'), \
                              private_key=settings.RECAPTCHA_PRIVATE_KEY, \
                              remoteip=request.META['REMOTE_ADDR'])
    
    if not resp.is_valid:
        data= {'error':True, 'Message': 'security text mismatch !!!'}
        return HttpResponse(json.dumps(data))
