<template>
	<section class="pop-in">
		<section>
			<section class="head">
				<figure class="icon font icon-lock"></figure>
				<figure class="title">{{$t('popins.fullscreen.securityCode')}}</figure>
				<figure class="subtitle">{{subtitle}}</figure>

				<br>
				<br>
				<Input placeholder="Security Code" type="password" centered="1" big="1" :text="code" v-on:changed="x => code = x" />
			</section>

		</section>

		<ActionBar :buttons-left="[{text:$t('generic.cancel'), click:() => returnResult(false)}]" :buttons-right="[{text:$t('generic.confirm'), red:true, click:() => returnResult(code)}]" />
	</section>
</template>

<script>
	import { mapActions, mapGetters, mapState } from 'vuex'
	import '../../../styles/popins.scss';
	import * as UIActions from "../../../store/ui_actions";

	export default {
		props:['popin'],
		data () {return {
			code:''
		}},
		computed:{
			subtitle(){
				return this.popin.data.props.subtitle;
			}
		},
		methods:{
			returnResult(code){
				this.popin.data.callback(code);
				this[UIActions.RELEASE_POPUP](this.popin);
			},

			...mapActions([
				UIActions.RELEASE_POPUP
			])
		},
	}
</script>

<style scoped lang="scss" rel="stylesheet/scss">
	@import "../../../styles/variables";

	.panel-container {
		overflow: auto;
		height: calc(100vh - 240px);
	}

	.auto-vote {
		max-width:600px;
		margin:0 auto;
		display:flex;

		.switch {

		}

		.details {
			padding-left:20px;
		}
	}

	.list {
		max-width:700px;
		margin:0 auto;
		width:100%;
		text-align:left;

	}

</style>
