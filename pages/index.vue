<template>
  <div class="bg-gray-100">
    <div class="h-screen flex items-center justify-center">
      <div class="flex flex-col rounded-lg shadow-lg w-2/3 p-10 bg-white">
        <div class="mb-4">
          <div v-if="device" class="flex flex-col space-y-4">
            <div class="flex flex-col">
              <span class="text-xs">Connected to:</span>
              <span class="font-bold" v-text="message" />
              <span v-text="error"/>
            </div>
            <div class="relative w-full appearance-none label-floating">
              <h6>Commands:</h6>
              <textarea
                v-model="code"
                class="
                  autoexpand
                  tracking-wide
                  py-2
                  px-4
                  leading-relaxed
                  appearance-none
                  block
                  w-full
                  rounded
                  outline-none
                  bg-gray-100
                  border-gray-200
                "
                rows="10"
                id="message"
                type="text"
                placeholder="TSPL/TSPL2 commands"
              ></textarea>
            </div>
          </div>
          <div v-else class="flex justify-center text-sm">
            <span>Please connect to a printer</span>
          </div>
        </div>
        <div class="flex space-x-2">
          <button
            v-if="device"
            class="bg-blue-600 text-gray-100 p-4 rounded flex-grow"
            :class="{
              'opacity-50 cursor-not-allowed': !code,
              'hover:bg-blue-700': code,
            }"
            @click="sendData"
            :disabled="!code"
          >
            Send
          </button>
          <button
            v-else
            class="
              hover:bg-blue-700
              bg-blue-600
              text-gray-100
              p-4
              rounded
              flex-grow
            "
            @click="connectPrinter"
          >
            Connect
          </button>
          <button
            v-if="device"
            class="hover:bg-gray-300 bg-gray-200 text-gray-700 p-4 rounded"
            @click="disconnectPrinter"
          >
            Disconnect
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
const VENDOR_ID = 4611;
const commands = `
SIZE 58 mm,30 mm
GAP 3 mm,0 mm
DIRECTION 1,0
CLS
QRCODE 24,24,Q,4,A,0,M2,S3,"a=1de5ef9a-896b-4384-8650-821f2524893c"
CODEPAGE UTF-8
TEXT 182,28,"2",0,1,1,"ID 2292"
TEXT 182,58,"2",0,1,1,"ІНВ 00001"
TEXT 182,88,"2",0,1,1,"Принтер"
TEXT 182,118,"2",0,1,1,"TSC TTP-244 Pro"
TEXT 24,210,"2.EFT",0,1,1,"https://my.unio24.com"
PRINT 1
`

export default {
  name: "IndexPage",
  data() {
    return {
      device: null,
      error: null,
      message: VENDOR_ID,
      code: commands,
      log: null
    };
  },
  methods: {
    async sendData() {
      const encoder = new TextEncoder();
      if (this.device) {
        
        var  encodeCommands =  encoder.encode(this.code)

        try {
          this.device.transferOut(1,encodeCommands)
        } catch (e) {
          this.error = "Device disconnected";
        }
      } else {
        this.error = "No device found";
      }
    },
    async connectPrinter() {
      this.error = null;
      console.log(window.navigator.userAgent);
  
      let devices = await navigator.usb.getDevices();
      this.device = devices[0];
      if (devices.length === 0) {
        try {
         this.device = await navigator.usb.requestDevice({filters: [{ vendorId: VENDOR_ID }],});
        } catch (e) {
          this.error = e;
        }
      }
      if (this.device) {
        await this.device.open();
        await this.device.claimInterface(0);
      } else {
        this.error = "No device found";
      }
    },
    async disconnectPrinter() {
      try{
        if (this.device) {
          await this.device.close();
          this.device = null;
        }
      } catch (e) {
        this.error = e
      }
    },
  },
};
</script>
