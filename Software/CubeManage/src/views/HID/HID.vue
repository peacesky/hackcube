<template>
  <div class="board">
    <cube-nav/>
    <div v-for="latest_crf_item in latest_crf_items" >
      <b-alert show variant="primary" dismissible>
        <h4 class="alert-heading">RF</h4>
        <p>
          In frequency {{latest_crf_item.freq}} found prot: {{latest_crf_item.prot}}, data: {{latest_crf_item.data}} signal.<u @click="clickRF">Learn More.</u>
        </p>
      </b-alert>
    </div>
    <b-alert :show="showNFCAlert" variant="success" dismissible>
      <h4 class="alert-heading">NFC</h4>
      <p>
        Found NFC card ID: {{latest_nfc_item.ID}}, <u @click="clickNFC">Learn More.</u>
      </p>
    </b-alert>
    <h1 class="text-center">Cube HID Manage</h1>
    <h3 class="text-center">Using Cube to simulate keyboard, mouse, HID and other devices</h3>
    <br/>
    <h5>Key to output:</h5>
    <b-form-input v-model="Key"
                  type="text"
                  placeholder="Key">
    </b-form-input>
    <br/>
    <div class="text-center">
      <Button type="ghost" icon="ios-cloud-upload-outline" @click="onClickSubmit">Submit</Button>
    </div>
    <br/><br/>


    <h5>Script</h5>
    <b-table :items="items" :fields="fields">
      <div slot="Run" slot-scope="data">
        <b-button size="sm" variant="primary" @click="onClick(data.index)">Run</b-button>
      </div>
    </b-table>
  </div>
</template>

<script>
    import CubeNav from '@/components/CubeNav';
    import axios from 'axios';
    import AxiosMixin from '../axios-mixin';

    export default {
      name: 'HID',
      components: {
        CubeNav,
      },
      mixins: [AxiosMixin],
      data() {
        return {
          Key: null,
          uploadHost: process.env.UPLOAD_API,
          extraData: { type: 'HID-Script' },
          fields: ['Info', 'Name', 'Run'],
          items: [
            { Index: 0, Info: '29d172a6', Name: 'Lock screen', Run: false },
            { Index: 1, Info: 'd2f392f1', Name: 'Add User', Run: false },
            { Index: 2, Info: '9209993f', Name: 'ShellCode', Run: false },
          ],
          latest_crf_items: [],
          showNFCAlert: false,
          latest_nfc_item: '',
        };
      },
      methods: {
        serialSend(parameter) {
          axios
            .get(`${process.env.BACKEND_HOST}/serial_send/${parameter}`)
            .then((response) => {
              const message = response.data.message;
              this.$Message.success(message);
            })
            .catch((err) => {
              if (err.response) {
                this.$Message.error(err.response.data.message);
              } else {
                this.$Message.error('Request fail');
              }
            });
        },
        onClickSubmit() {
          if (this.Key) {
            this.serialSend(`h${this.Key}`);
          } else {
            this.$Message.error('Please input Key before submit.');
          }
        },
        onClick(index) {
          switch (index) {
            case 0:
              this.serialSend('ha');
              break;
            case 1:
              this.serialSend('hb');
              break;
            case 2:
              // 暂时不管
              break;
            default:
              break;
          }
        },
        onUploadSuccess() {
          this.$Message.success('Upload success');
        },
        onUploadFail() {
          this.$Message.error('Upload fail');
        },
      },
      timers: {
        fetchCRFItems: { time: 3400, autostart: true, repeat: true },
        fetchNFCData: { time: 3000, autostart: true, repeat: true },
      },
    };
</script>

<style scoped>

</style>
