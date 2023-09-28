<template>
  <div id="main">
    <h3>Window Control</h3>
    <b>isExpanded</b>: {{ TWA.isExpanded }}
    <button @click="TWA.expand()">Expand</button>
    <button @click="TWA.close()">Close</button><br>

    <b>isClosingConfirmationEnabled</b>: {{ TWA.isClosingConfirmationEnabled }}
    <button @click="toggleClosingDialog()">Enable/Disable Confirmation Dialog</button><br>

    <b>viewportHeight</b>: {{ TWA.viewportHeight }} <br>
    <b>viewportStableHeight</b>: {{ TWA.viewportStableHeight }} <br>


    <h3>Theme colors</h3>
    <b>colorScheme</b>: {{ TWA.colorScheme }} <br>
    <b>themeParams</b>:<br>
    <div v-for="(rgb_color, key) in TWA.themeParams">
     <label for="head"><b>{{ key }}</b>: {{ rgb_color }}
       <input type="color" :value="rgb_color" disabled>
     </label> <br>
    </div>

    <h3>Web App colors</h3>
     <label for="header"><b>headerColor</b>: {{ TWA.headerColor }}
       <input type="color" name="header" :value="TWA.headerColor" disabled><br>
       <select @input="changeHeaderColor">
         <option disabled value="">Please select one</option>
         <option>bg_color</option>
         <option>secondary_bg_color</option>
       </select>
     </label> <br>

     <label for="background"> <b>backgroundColor</b>: {{ TWA.backgroundColor }}
       <input type="color" name="background" :value="TWA.backgroundColor" disabled><br>
       <select @input="changeBackgroundColor">
         <option disabled value="">Please select one</option>
         <option>bg_color</option>
         <option>secondary_bg_color</option>
       </select>
     </label> <br>

    <h3>Data received</h3>
    <b>initData</b>: {{ TWA.initData }} <br>
    <b>initDataUnsafe</b>: <pre>{{ TWA.initDataUnsafe }}</pre><br>
    <b>platform</b>: <pre>{{ TWA.platform }}</pre><br>
 

    <h3>Bot API version available</h3>
    <b>version</b>: {{ TWA.version }} <br>
    <b>isVersionAtLeast('6.9')</b>: {{ TWA.isVersionAtLeast('6.9') }} <br>

    <h3>Haptic Feedback</h3>
    <select v-model="style_selected">
      <option disabled value="">Please select one</option>
      <option>light</option>
      <option>medium</option>
      <option>heavy</option>
      <option>rigid</option>
      <option>soft</option>
    </select>
    <button @click="hapticImpact()">Haptic Feedback ({{ style_selected }})</button><br>

    <h3>Functions and buttons</h3>
    <button @click="TWA.openLink('https://github.com/MBoretto/telegram-bot-vue-wep-app')">Open link in an external browser</button><br>
    <button @click="toggleBackButton()">Show/hide Back Button</button><br>
    <button @click="toggleMainButton()">Show/hide Main Button</button>
    <button @click="toggleEnableMainButton()">Enable/Disable Main Button</button>
    <button @click="toggleProgressMainButton()">Show/Hide Main Button progress</button><br>

    <h3>QR scanner</h3>
    <button @click="showQRScanner()">Scan QR code</button><br>
    <h3>Clipboard</h3>
    <button @click="TWA.readTextFromClipboard()">Press to read the clipboard</button> <br> {{ clipboard }}<br>
    <h3>Popups</h3>
    <button @click="TWA.showAlert('Showing an Alert!!')">Show Alert</button><br>
    <button @click="TWA.showConfirm('Showing confirm message')">Show Confirm</button><br>
    <button @click="showPopup()">Show Popup message</button><br>
    <button @click="showPopup2()">Show Popup message2</button><br>
    <br>
    <h3>Cloud Storage</h3>
 


    <input type="text" id="key" name="key" placeholder="key" v-model="akey">
    <input type="text" id="value" name="value" placeholder="value" v-model="avalue">
    <button @click="addKeyValue()">Set key</button><br>

    
    <div v-for="key in cloud_storage_keys">
      - {{ key }} <button @click="removeKey(key)">Delete</button><br>
    </div>

    <br>

    <button @click="TWA.CloudStorage.getKeys(this.processKeys)">Get keys</button><br>


    <div v-for="value in cloud_storage_values">
      - {{ value }} <br>
    </div>
    <button @click="TWA.CloudStorage.getItems(cloud_storage_keys, this.processItems)">Get Values</button><br>

  </div>
</template>


<script>
// Objects
//BackButton
//MainButton
//HapticFeedback

// Methods
//enableClosingConfirmation() NEW
//disableClosingConfirmation() NEW
//onEvent(eventType, eventHandler)

//offEvent(eventType, eventHandler)
//sendData(data)
//openTelegramLink(url)
//openInvoice(url[, callback])
//showPopup(params[, callback]) NEW
//showAlert(message[, callback]) NEW
//showConfirm(message[, callback]) NEW
export default {
  data() {
    return {
      style_selected: 'medium',
      clipboard: null,
      cloud_storage_keys: [],
      cloud_storage_values: [],
      akey: null,
      avalue: null,
    };
  },
  computed: {
    arrayAsJSON() {
      // Convert the array to a JSON string
      return JSON.stringify(this.cloud_storage_values, null, 2);
    }
  },
  created() {
    // Binnding function to all the event types
    this.TWA.onEvent('themeChanged', this.themeChanged);
    // triggered too often
    //this.TWA.onEvent('viewportChanged', this.viewportChanged);
    this.TWA.onEvent('mainButtonClicked', this.mainButtonClicked);
    this.TWA.onEvent('backButtonClicked', this.backButtonClicked);
    // I couldn't trigger this yet
    this.TWA.onEvent('settingsButtonClicked', this.settingsButtonClicked);
    this.TWA.onEvent('invoiceClosed', this.invoiceClosed);
    // Seems that the popup is an alter itself
    // Commenting this otherwise I'm stuck in a loop
    //this.TWA.onEvent('popupClosed', this.popupClosed);
    this.TWA.onEvent('qrTextReceived', this.processQRCode);
    this.TWA.onEvent('clipboardTextReceived', this.processClipboard);
    
  },
  mounted() {
    // What is the best? mounted or created??
    this.TWA.ready();
  },
  methods: {
    // attached with onEvent function during created
    themeChanged() {
      this.TWA.showAlert('Theme has changed');
    },
    viewportChanged() {
      this.TWA.showAlert('Viewport has changed');
    },
    mainButtonClicked() {
      this.TWA.showAlert('Main button was pressed');
      window.Telegram.WebApp.showAlert('Main button was pressed version2');
    },
    backButtonClicked() {
      this.TWA.showAlert('back button was clicked');
    },
    settingsButtonClicked() {
      this.TWA.showAlert('Settings button pressed');
    },
    invoiceClosed() {
      this.TWA.showAlert('Invoice was closed');
    },
    popupClosed() {
      this.TWA.showAlert('Popup was closed');
    },
    processQRCode(data) {
       this.TWA.closeScanQrPopup();
       this.TWA.showAlert(data.data);
    },
    processClipboard(data) {
      if (data.data === null) {
        this.clipboard = "The Web App has no access to the clipboard"
        return;
      }
      if (data.data == '') {
        this.clipboard = "Clipboard content is not a string"
        return;
      }
      this.clipboard = data.data;
    },
    // End of callbacks
    //Cloud Storage
    addKeyValue() {
      if (this.akey === null || this.akey === '') {
        this.TWA.showAlert('Key is empty');
        return;
      }
      // check if the key is longer than 128 characters
      if (this.akey.length > 128) {
        this.TWA.showAlert('Error Key is longer than 128 characters');
        return;
      }
      // check if the key contains a space
      if (this.akey.indexOf(' ') >= 0) {
        this.TWA.showAlert('Error Key contains a space');
        return;
      }
      // check if the value is longer than 4096 characters
      if (this.avalue.length > 4096) {
        this.TWA.showAlert('Error Value is longer than 4096 characters');
        return;
      }
      this.TWA.CloudStorage.setItem(this.akey, this.avalue);
      this.TWA.showAlert('Item added key: ' + this.akey + ' value: ' + this.avalue);
    },
    removeKey(key) {
      this.TWA.CloudStorage.removeItem(key);
    },
    processKeys(error, data) {
      if (error) {
        this.TWA.showAlert(error);
        return;
      }
      this.cloud_storage_keys = data;
    },
    processItems(error, data) {
      if (error) {
        this.TWA.showAlert(error);
        return;
      }
      this.cloud_storage_values = data;
      const mergedArray = keys.map((key, index) => ({ [key]: values[index] }));


    },
    // End of Cloud Storage
    showQRScanner() {
      const par = {
          text: "Press to scan"
        };
      this.TWA.showScanQrPopup(par);
    },
    changeHeaderColor(event) {
        const color = event.target.value;
        this.TWA.setHeaderColor(color);
    },
    changeBackgroundColor(event) {
        const color = event.target.value;
        this.TWA.setBackgroundColor(color);
    },
    toggleBackButton() {
      if (this.TWA.BackButton.isVisible) {
        this.TWA.BackButton.hide();
      } else {
        this.TWA.BackButton.show();
      }
    },
    toggleMainButton() {
      if (this.TWA.MainButton.isVisible) {
        this.TWA.MainButton.hide();
      } else {
        this.TWA.MainButton.show();
      }
    },
    toggleEnableMainButton() {
      if (!this.TWA.MainButton.isVisible) {
        this.TWA.MainButton.show();
      }
      if (this.TWA.MainButton.isActive) {
        this.TWA.MainButton.disable();
      } else {
        this.TWA.MainButton.enable();
      }
    },
    toggleProgressMainButton() {
      if (!this.TWA.MainButton.isVisible) {
        this.TWA.MainButton.show();
      }
      if (this.TWA.MainButton.isActive) {
        this.TWA.MainButton.hideProgress();
      } else {
        this.TWA.MainButton.showProgress();
      }
    },
    toggleClosingDialog() {
      if (this.TWA.isClosingConfirmationEnabled) {
        this.TWA.disableClosingConfirmation();
      } else {
        this.TWA.enableClosingConfirmation();
      }
    },
    hapticImpact() {
      this.TWA.HapticFeedback.impactOccurred(this.style_selected);
    },
    showPopup() {
      const par = {
                    title: "Popup title",
                    message: "Popup with default, ok and close buttons",
                    buttons: [
                      {id: "default", type: "default", text: "default"},
                      {id: "ok", type: "ok", text: "ok"},
                      {id: "close", type: "close", text: "close"}
                    ]
                  };

      this.TWA.showPopup(par);
    },
    showPopup2() {
      const par = {
                    title: "Popup title",
                    message: "Popup with cancel and destrucitve buttons",
                    buttons: [
                      {id: "cancel", type: "cancel", text: "cancel"},
                      {id: "destructive", type: "destructive", text: "destructive"}
                    ]
                  };

      this.TWA.showPopup(par);
    }
  }
}
</script>

<style scoped>
/*
bg_color            .
secondary_bg_color  var(--tg-theme-secondary-bg-color)
link_color          var(--tg-theme-link-color).
*/
#main {
  background-color: var(--tg-theme-bg-color, white);
  color: var(--tg-theme-text-color, black);
  word-wrap: break-word;
}
b {
  color: var(--tg-theme-hint-color, black);
}
h3 {
  color: var(--tg-theme-text-color, black);
}
button {
  background-color: var(--tg-theme-button-color, #008CBA);
  border: 5px;
  color: var(--tg-theme-button-text-color, black);
  padding: 15px;
  margin: 5px;
  text-align: center;
  text-decoration: none;
  display: inline-block;
  font-size: 15px;
}
</style>
