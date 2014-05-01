Strap is a real-time wearable analytics platform for developers. This repository contains the SDK for Pebble. Strap is currently in private beta, and you'll need an account on the dashboard to use this SDK. Signup today at http://www.straphq.com/register.


##Strap Pebble SDK Quick Start Guide


We currently support the Pebble Javascript SDK (>2.0). Strap utilizes AppMessage to communicate between the watch and the connected phone, and tries to be smart about how often it sends data in order to preserve battery life. 

We hope that getting started with the Strap SDK is easy for you. These steps shouldn't take more than 15-20 minutes. Grab a cup of coffee, and let's get to it!

---
1. Login to the <a href="http://www.straphq.com/login">Strap Dashboard</a> and create an app. You'll need your App ID handy for the next step.

2. Paste the JS code into your pebble-app.js. This step is important, because without it the Strap code on the device can't communicate with the Strap API. This step has two parts: 
  * Paste the following initialization code at the top of your Javascript file. 
    ```
    // ------------------------------
    //  Start of Strap API
    // ------------------------------
    var strap_api_num_samples=10;var strap_api_url="https://api.straphq.com/create/visit/with/";var strap_api_timer_send=null;var strap_api_const={};strap_api_const.KEY_OFFSET=48e3;strap_api_const.T_TIME_BASE=1e3;strap_api_const.T_TS=1;strap_api_const.T_X=2;strap_api_const.T_Y=3;strap_api_const.T_Z=4;strap_api_const.T_DID_VIBRATE=5;strap_api_const.T_ACTIVITY=2e3;strap_api_const.T_LOG=3e3;var strap_api_can_handle_msg=function(data){var sac=strap_api_const;if((sac.KEY_OFFSET+sac.T_ACTIVITY).toString()in data){return true}if((sac.KEY_OFFSET+sac.T_LOG).toString()in data){return true}return false};var strap_api_clone=function(obj){if(null==obj||"object"!=typeof obj)return obj;var copy={};for(var attr in obj){if(obj.hasOwnProperty(attr))copy[attr]=obj[attr]}return copy};var strap_api_log=function(data,min_readings,log_params){var sac=strap_api_const;var lp=log_params;if(!((sac.KEY_OFFSET+sac.T_LOG).toString()in data)){var convData=strap_api_convAcclData(data);var tmpstore=window.localStorage["strap_accl"];if(tmpstore){tmpstore=JSON.parse(tmpstore)}else{tmpstore=[]}tmpstore=tmpstore.concat(convData);if(tmpstore.length>min_readings){window.localStorage.removeItem("strap_accl");var req=new XMLHttpRequest;req.open("POST",strap_api_url,true);var tz_offset=(new Date).getTimezoneOffset()/60*-1;var query="app_id="+lp["app_id"]+"&resolution="+(lp["resolution"]||"")+"&useragent="+(lp["useragent"]||"")+"&action_url="+"STRAP_API_ACCL"+"&visitor_id="+(lp["visitor_id"]||Pebble.getAccountToken())+"&visitor_timeoffset="+tz_offset+"&accl="+encodeURIComponent(JSON.stringify(tmpstore))+"&act="+(tmpstore.length>0?tmpstore[0].act:"UNKNOWN");req.setRequestHeader("Content-type","application/x-www-form-urlencoded");req.setRequestHeader("Content-length",query.length);req.setRequestHeader("Connection","close");req.onload=function(e){if(req.readyState==4&&req.status==200){if(req.status==200){console.log("Sent")}else{console.log("Error")}}};req.send(query)}else{window.localStorage["strap_accl"]=JSON.stringify(tmpstore)}}else{var req=new XMLHttpRequest;req.open("POST",strap_api_url,true);var tz_offset=(new Date).getTimezoneOffset()/60*-1;var query="app_id="+lp["app_id"]+"&resolution="+(lp["resolution"]||"")+"&useragent="+(lp["useragent"]||"")+"&action_url="+data[(sac.KEY_OFFSET+sac.T_LOG).toString()]+"&visitor_id="+(lp["visitor_id"]||Pebble.getAccountToken())+"&visitor_timeoffset="+tz_offset;req.setRequestHeader("Content-type","application/x-www-form-urlencoded");req.setRequestHeader("Content-length",query.length);req.setRequestHeader("Connection","close");req.onload=function(e){if(req.readyState==4&&req.status==200){if(req.status==200){console.log("Sent")}else{console.log("Error")}}};req.send(query)}};var strap_api_convAcclData=function(data){var sac=strap_api_const;var convData=[];var time_base=parseInt(data[(sac.KEY_OFFSET+sac.T_TIME_BASE).toString()]);for(var i=0;i<strap_api_num_samples;i++){var point=sac.KEY_OFFSET+10*i;var ad={};var key=(point+sac.T_TS).toString();ad.ts=data[key]+time_base;key=(point+sac.T_X).toString();ad.x=data[key];key=(point+sac.T_Y).toString();ad.y=data[key];key=(point+sac.T_Z).toString();ad.z=data[key];key=(point+sac.T_DID_VIBRATE).toString();ad.vib=data[key]=="1"?true:false;ad.act=data[(sac.KEY_OFFSET+sac.T_ACTIVITY).toString()];convData.push(ad)}return convData};
    
    // ------------------------------
    //  End of Strap API
    // ------------------------------
    ```
  * Paste the following Strap AppMessage handler in your AppMessage event listener. Be sure to insert your App ID in this step! **Important: If you already have an appmessage event listener, you don't need to register it again. Just take the code inside of this function and add it to your existing event listener.**
  ```
    Pebble.addEventListener("appmessage",
        function(e) {
            // Strap API: Developer updates these parameters to fit
            var strap_params = {
                // *** change the app id! *** //
                app_id: "xyzabc123abcxyz",
                resolution: "144x168",
                useragent: "PEBBLE/2.0"
            };
    
            // -------------------------
            //  Strap API inclusion in appmessage
            //  This allows Strap to process Strap-related messages
            //  DO NOT EDIT
            // -------------------------
            if(strap_api_can_handle_msg(e.payload)) {
                clearTimeout(strap_api_timer_send);
                var params = strap_api_clone(strap_params);
                strap_api_log(e.payload, 200, params);
                strap_api_timer_send = setTimeout(function() {
                    strap_api_log({}, 0, params);
                }, 10 * 1000);
            }
            // -------------------------
              
        }
    );
    ```

3. Include the Strap C source in your src directory. We typically lean towards a directory structure that looks like this:

    ```
    /pebble-app
    - /src
    - - / js
    - - - pebble-js-app.js (put Strap JS code in here)
    - - / strap (the Strap C source)
    - - - strap.c
    - - - strap.h
    - - pebble-app.c
    ```
4. Include strap.h in any of your source that contains actions you want to track in Strap. At a minimum, you must include the Strap header and call the init functions below for Strap to track anything. Make sure you correct the path relative to your source.

    `# include "strap/strap.h"`

5. Initialize Strap in your Pebble code

    In a typical Pebble pattern, your main() calls an init() and deinit() function. Here, you'll need to include the strap_init() and strap_deinit() functions, respectively. **Important: Your code must call app_message_open for Strap to communicate!**
    ```
    static void init(void) {
      
      window = window_create();
      window_set_click_config_provider(window, click_config_provider);
      window_set_window_handlers(window, (WindowHandlers) {
        .load = window_load,
        .unload = window_unload,
      });
      const bool animated = true;
      window_stack_push(window, animated);
      
      app_message_register_inbox_received(in_received_handler);
      
      int in_size = app_message_inbox_size_maximum();
      int out_size = app_message_outbox_size_maximum();
      app_message_open(in_size, out_size);
    
      // initialize strap
      strap_init();
    }
    
    static void deinit(void) {
      // unload strap
      strap_deinit();
      
      window_destroy(window);
    }
    
    int main(void) {
      init();
      app_event_loop();
      deinit();
    }
    ```

6. Start tracking actions! 
    ```
    static void select_button_click_handler(ClickRecognizerRef recognizer, void *context) {
      // do something on your Pebble
      strap_log_action("/select");
    }
    ```

If you're stuck, check out the sample project that should help you get started.

Accelerometer data is automatically sent in the background periodically and crunched on the server every few hours. If you have issues or problems, please open an issue on GitHub or visit our community support portal at http://strap.uservoice.com

