<template>
	<section class="center-panel">
		<h2>{{$t('panels.keypair.scanQR')}}</h2>
		<p></p>

		<br>
		<!-- Note, this won't show up on non https or localhost when inside of a mobile container. -->
		<qrcode-stream v-if="!refresh" @decode="qrScanned"></qrcode-stream>

		<ActionBar v-if="returnOnly" :buttons-left="[{text:$t('generic.back'), click:() => $emit('back')}]" :buttons-right="[{text:$t('generic.skip'), click:() => $emit('next')}]" />
	</section>
</template>

<script>
	import {mapGetters} from 'vuex';
	import KeyPairService from "@walletpack/core/services/secure/KeyPairService";
	import Keypair from "@walletpack/core/models/Keypair";
	import PopupService from "../../../services/utility/PopupService";
	import {Popup} from "../../../models/popups/Popup";
	import QRService from "@walletpack/core/services/secure/QRService";

	export default {
		props:['returnOnly'],
		data(){return {
			refresh:false
		}},
		computed:{
			...mapGetters([
				'keypairs'
			])
		},
		methods:{
			qrScanned(qr){
				const {data, salt} = JSON.parse(qr);
				PopupService.push(Popup.verifyPassword(async pass => {
					if(!pass) return;
					this.setWorkingScreen(true);
					const privateKey = await QRService.decryptQR(data, salt, pass).catch(() => null);
					if(!privateKey){
						this.refresh = true;
						this.$nextTick(() => this.refresh = false);
						this.setWorkingScreen(false);
						return PopupService.push(Popup.snackbar(this.$t('errors.badQrDecryption')));
					}
					const keypair = Keypair.placeholder();
					keypair.privateKey = privateKey;
					await KeyPairService.makePublicKeys(keypair);
					this.setWorkingScreen(false);

					if(!keypair.isUnique()){
						this.setWorkingScreen(false);
						return PopupService.push(Popup.snackbar(this.$t('errors.keypairExists')));
					}
					this.$emit('key', keypair);
				}, true))
			}
		},
		watch:{
			['privateKey'](){
				this.$emit('key', this.privateKey.trim().replace(/\W/g, '').replace('0x', ''));
			}
		}
	}
</script>

<style scoped lang="scss">

</style>
