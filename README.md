# Docs SDK WEB client Callcenter Bizfly


[![N|Solid](https://cldup.com/dTxpPi9lDf.thumb.png)](https://nodesource.com/products/nsolid)


## Features SDK 

- Get Configure Extension 
- Create SIP stack 
- Make A call
- Receive A Call
- Tranfer
- Hang up (Reject)
- UnRegister




## Step 1: Adding the Bizfly Cllcenter SDK
 - Add js files to Header and set 3 params API_KEY, EXT_NUMBER, HOTLINE  :
```
    <!-- SIPML5 API:
        RELEASE VERSION: 'SIPml-api.js'
    -->
    <script src="https://api-stagging-callcenter.bizflycloud.vn/webphone_crm/assets/SIPml-api.js" type="text/javascript"> </script>


 <!-- JAVASCRIPT API-->
    <script type="text/javascript">
        var API_KEY = "xxxxxxxxxxxxxxxxxxx";
        var EXT_NUMBER = "xxxxx";
        var HOTLINE = "xxxxxxxxxxxx";
    </script>
```
## Step 2: Load Configure and Connecting to Bizfly Callcenter Server
  - Function get configure Extension and Save to local Storage
      ```
      get_configure(API_KEY, EXT_NUMBER, HOTLINE) ;
      ```
 
 
         
  - Callback function for SIP Stacks
    ```
       // Callback function for SIP Stacks
        function onSipEventStack(e /*SIPml.Stack.Event*/) {
            console.log('==stack event = ' + e.type);
            case 'starting':default:break ;
        }
    ```
    * event started ( e.type = "started" )
    ```
      // LogIn (REGISTER) as soon as the stack finish starting
      oSipSessionRegister = this.newSession('register', {
                                expires: 200,
                                events_listener: { events: '*', listener: onSipEventSession },
                                sip_caps: []
                            });
        oSipSessionRegister.register();
        
    Example :
    case 'started':
                    {
                        // catch exception for IE (DOM not ready)
                        try {
                            // LogIn (REGISTER) as soon as the stack finish starting
                            oSipSessionRegister = this.newSession('register', {
                                expires: 200,
                                events_listener: { events: '*', listener: onSipEventSession },
                                sip_caps: [
                                            { name: '+g.oma.sip-im', value: null },
                                            //{ name: '+sip.ice' }, // rfc5768: FIXME doesn't work with Polycom TelePresence
                                            { name: '+audio', value: null },
                                            { name: 'language', value: '\"en,fr\"' }
                                ]
                            });
                            oSipSessionRegister.register();
                        }
                        catch (e) {
                          // Eror log in login
                           console.log("Error" + e );
                        }
                        break;
                    }
                       
    ```
    * event incoming audio (e.type == 'i_new_call')  
    
    ```
       if(e.type == 'i_new_call'){ // incoming audio
                    acceptCall(e);
                }
       Example :
       
        case 'i_new_call':
                    {
                        if (oSipSessionCall) {
                            // do not accept the incoming call if we're already 'in call'
                            e.newSession.hangup(); // comment this line for multi-line support
                        }
                        else {
                            oSipSessionCall = e.newSession;
                            // start listening for events
                            oSipSessionCall.setConfiguration(oConfigCall);

                            
                            startRingTone();
                            
                            // Phone number from incoming 
                            var sRemoteNumber = (oSipSessionCall.getRemoteFriendlyName() || 'unknown');
                                console.log("Incooming from "+ sRemoteNumber)
                            // Show notify 
                            showNotifICall(sRemoteNumber);
                        }
                        break;
                    }

                
    ```
  - Callback function for SIP sessions (INVITE, REGISTER, ...)
    ```
        function onSipEventSession(e /* SIPml.Session.Event */) {
            console.log('==session event = ' + e.type);
        }       
    ```
     * event connected (e.type =connecting|connected)
    ```
    Exmple: 
        case 'connecting': case 'connected':
                    {
                        var bConnected = (e.type == 'connected');
                        if (e.session == oSipSessionRegister) {
                           console.log(e.description);
                           
                        }
                        else if (e.session == oSipSessionCall) {
                             console.log(e.description);
                        }
                        break;
                    } // 'connecting' | 'connected'
    ```
    * event terminal (e.type = case 'terminating': case 'terminated':)
    ```
    Example
      case 'terminating': case 'terminated':
                    {
                        if (e.session == oSipSessionRegister) {
                          console.log(e.description);
                        }
                        else if (e.session == oSipSessionCall) {
                            console.log(e.description);
                        }
                        break;
                    } // 'terminating' | 'terminated'
    ```
   - In order to connect to Bizfly Callcenter Server, instantiate a BizflyClient instance:
    ```
    //check isWebRTCSupported
    console.log('SIPml.isWebRtcSupported : ' + SIPml.isWebRtcSupported());
    //check WebSocket Support
    console.log('SIPml.isWebSocketSupported:' + SIPml.isWebSocketSupported());
      ```
  - Initialize the engine and define params global 
        ```
          SIPml.init(<function callback>);
          
          Example Code:
            var oRingTone, oRingbackTone;
            var oSipStack, oSipSessionRegister, oSipSessionCall, oSipSessionTransferCall;
            var videoRemote, videoLocal, audioRemote;
           function postInit() {
            // check for WebRTC support
            // checks for WebSocket support
             oConfigCall = {
                audio_remote: audioRemote,
                bandwidth: { audio: undefined, video: undefined },
                events_listener: { events: '*', listener: onSipEventSession },
                sip_caps: [
                                { name: '+g.oma.sip-im' },
                                { name: 'language', value: '\"en,fr\"' }
                ]
            };
           }
            // initialize SIPML5
                SIPml.init(postInit);
                
         ```
    
  - Create SIP stack Register Connect to Bizfly Callcenter server 
  ```
          var sipStack;
            var eventsListener = function(e){
                if(e.type == 'started'){
                    login();
                }
                else if(e.type == 'i_new_call'){ // incoming audio/video call
                    acceptCall(e);
                }
            }
            
            
            function createSipStack(){
                sipStack = new SIPml.Stack({
                        realm: 'example.org', // mandatory: domain name
                        impi: 'bob', // mandatory: authorization name (IMS Private Identity)
                        impu: 'sip:bob@example.org', // mandatory: valid SIP Uri (IMS Public Identity)
                        password: 'mysecret', // optional
                        display_name: 'Bob legend', // optional
                        websocket_proxy_url: 'wss://sipml5.org:10062', // optional
                        outbound_proxy_url: 'udp://example.org:5060', // optional
                        enable_rtcweb_breaker: false, // optional
                        events_listener: { events: '*', listener: eventsListener }, // optional: '*' means all events
                        sip_headers: [ // optional
                                { name: 'User-Agent', value: 'IM-client/OMA1.0 sipML5-v1.0.0.0' },
                                { name: 'Organization', value: 'Doubango Telecom' }
                        ]
                    }
                );
            }
            sipStack.start();
    
    Example Code :
   // create SIP stack
      oSipStack = new SIPml.Stack({
          realm: (window.localStorage ? window.localStorage.getItem('bizfly.callcenter.identity.realm') : null),
          impi: (window.localStorage ? window.localStorage.getItem('bizfly.callcenter.identity.impi') : null),
          impu: (window.localStorage ? window.localStorage.getItem('bizfly.callcenter.identity.impu') : null),
          password: (window.localStorage ? window.localStorage.getItem('bizfly.callcenter.identity.password') : null),
          display_name: (window.localStorage ? window.localStorage.getItem('bizfly.callcenter.identity.display_name') : null),
          websocket_proxy_url: (window.localStorage ? window.localStorage.getItem('bizfly.callcenter.identity.websocket') : null),
          outbound_proxy_url: (window.localStorage ? window.localStorage.getItem('bizfly.callcenter.expert.sip_outboundproxy_url') : null),
          events_listener: { events: '*', listener: <Function receive Event> },
           sip_headers: [
               { name: 'User-Agent', value: 'IM-client/OMA1.0 sipML5-v1.2016.03.04' },
                { name: 'Organization', value: 'Doubango Telecom' }
                    ]
                }
                );
        if (oSipStack.start() != 0) {
                   // Failed to start the SIP stack
                }
                else {
                    // Login Success
                       return ;
                }
                
  ```

## Make A Call
 ```
            
            var callSession;
            var eventsListener = function(e){
                console.info('session event = ' + e.type);
            }
            var makeCall = function(){
                callSession = sipStack.newSession('call-audio', {
                    audio_remote: document.getElementById('audio-remote'),
                    events_listener: { events: '*', listener: eventsListener } // optional: '*' means all events
                });
                callSession.call('phone_number');
            }
        

To accept incoming audio/video call:
            var acceptCall = function(e){
                e.newSession.accept(); // e.newSession.reject() to reject the call
            }
    
Example :
var  oConfigCall = {
                audio_remote: audioRemote,
                video_local: viewVideoLocal,
                video_remote: viewVideoRemote,
                screencast_window_id: 0x00000000, // entire desktop
                bandwidth: { audio: undefined, video: undefined },
                video_size: { minWidth: undefined, minHeight: undefined, maxWidth: undefined, maxHeight: undefined },
                events_listener: { events: '*', listener: onSipEventSession },
                sip_caps: [
                                { name: '+g.oma.sip-im' },
                                { name: 'language', value: '\"en,fr\"' }
                ]
            };
            
   // makes a call (SIP INVITE), call audio : s_type=call_audio
        function sipCall(s_type, phone_number) {
            if (oSipStack && !oSipSessionCall && !tsk_string_is_null_or_empty(txtPhoneNumber.value)) {
                // create call session
                oSipSessionCall = oSipStack.newSession(s_type, oConfigCall);
                // make call
                if (oSipSessionCall.call(phone_number) != 0) {
                    oSipSessionCall = null;
                   console.log('Failed to make call');
                    return;
                }
            }
            else if (oSipSessionCall) {
               console.log('<i>Connecting...</i>');
                oSipSessionCall.accept(oConfigCall);
            }
        }
       --> Maake call sipCall('call_audio', '0834771233') 
       --> Answer incoming  sipCall ('call_audio', "");
 ```

 ## Hangup
  ```
  oSipSessionCall.hangup({ events_listener: { events: '*', listener: <function callback event > } }); 
  
  Example :
          // terminates the call (SIP BYE or CANCEL)
        function sipHangUp() {
            if (oSipSessionCall) {
               Console.log('<i>Terminating the call...</i>');
                oSipSessionCall.hangup({ events_listener: { events: '*', listener: onSipEventSession } });
            }
        }
        
   ```
   
    ##  Tranfer 
   ```
   oSipSessionCall.transfer(phone number)
   
   Example:
    // transfers the call
        function sipTransfer(s_destination) {
            if (oSipSessionCall) {
                if (!tsk_string_is_null_or_empty(s_destination)) {
                    btnTransfer.disabled = true;
                    if (oSipSessionCall.transfer(s_destination) != 0) {
                        console.log('<i>Call transfer failed</i>');
                     
                        return;
                    }
                     console.log( '<i>Transfering the call...</i>');
                }
            }
        }
   
   ```    
      ##  Unregiter
         ```    
         oSipStack.stop(); // shutdown all sessions
         window.localStorage.clear(); // Remove configure extension
         Example:
           // sends SIP REGISTER (expires=0) to logout
        function sipUnRegister() {
            if (oSipStack) {
                oSipStack.stop(); // shutdown all sessions
                window.localStorage.clear();
            }
            return "Logout Success";
        }
            ```    
            
    ## Demo 
    https://api-stagging-callcenter.bizflycloud.vn/webphone_crm/call.html
   
   
 
