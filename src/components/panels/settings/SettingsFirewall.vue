<template>
    <section>

        <section class="action-box top-pad">
            <label>{{$t('settings.firewall.ridl.title')}}</label>
            <p>{{$t('settings.firewall.ridl.description')}}</p>

            <Switcher :state="scatter.settings.firewall" @click.native="toggleFirewall" />
        </section>

        <section class="action-box top-pad">
            <label>{{$t('settings.firewall.blocker.title')}}</label>
            <p>{{$t('settings.firewall.blocker.description')}}</p>
            <br>
            <br>

            <label>Add a new restricted action</label>
            <section class="split-inputs">
                <Select start-left="1" bordered="1"
                        :options="blockchains"
                        :parser="x => blockchainName(x)"
                        :selected="blockchain"
                        v-on:selected="x => blockchain = x" />
                <Input placeholder="Contract" :text="newContract" v-on:changed="x => newContract = x.trim().toLowerCase()" />
                <Input placeholder="Action" :text="newAction" v-on:changed="x => newAction = x.trim().toLowerCase()" />
            </section>
            <br>
            <Button blue="1" text="Add Restriction" @click.native="addBlacklistedAction" />
            <br>
            <br>
            <hr>
            <label>{{$t('settings.firewall.blocker.blacklisted')}}</label>
            <br>
            <section class="blacklisted-actions">
                <section class="blacklist" v-for="(actions, contract) in blacklistActions">
                    <figure class="blockchain">{{blockchainName(contract.split('::')[0])}}</figure>
                    <figure class="contract">{{contract.split('::')[1]}}</figure>
                    <figure class="actions">
                        <figure class="action" v-for="action in actions">
                            <figure class="action-name">{{action}}</figure>
                            <Button @click.native="removeBlacklist(contract, action)" class="remove" :text="$t('generic.remove')" small="1" />
                        </figure>
                    </figure>
                </section>
            </section>
        </section>

    </section>
</template>

<script>
    import { mapActions, mapGetters, mapState } from 'vuex'
    import * as Actions from '@walletpack/core/store/constants';
    import {Blockchains, BlockchainsArray} from "@walletpack/core/models/Blockchains";

    export default {
        data () {return {
        	newContract:'',
            newAction:'',
	        blockchains:BlockchainsArray.map(x => x.value),
            blockchain:Blockchains.EOSIO,
        }},
        computed:{
            ...mapState([
                'scatter',
            ]),
            ...mapGetters([
                'blacklistActions',
            ]),
        },
        methods: {
	        toggleFirewall(){
		        const scatter = this.scatter.clone();
		        scatter.settings.firewall = !scatter.settings.firewall;
		        this[Actions.SET_SCATTER](scatter);
            },
            addBlacklistedAction(){
                const scatter = this.scatter.clone();
                scatter.settings.blacklistAction(this.blockchain, this.newContract, this.newAction);
                this[Actions.SET_SCATTER](scatter);
            },
            removeBlacklist(blockchainAndContract, action){
	        	const [blockchain, contract] = blockchainAndContract.split('::');
	            const scatter = this.scatter.clone();
	            scatter.settings.removeBlacklistedAction(blockchain, contract, action);
	            this[Actions.SET_SCATTER](scatter);
            },
            ...mapActions([
                Actions.SET_SCATTER
            ])
        },
    }
</script>

<style scoped lang="scss" rel="stylesheet/scss">
    @import "../../../styles/variables";

    .split-inputs {
        .input {
            margin-bottom:0;

            @media (max-width: $breakpoint-mobile) {
              margin-bottom:10px;
            }
        }
    }

    .blacklisted-actions {
        display:flex;
        flex-direction: column;


        .blacklist {
            padding:10px;
            border:1px solid $blue;
            border-radius:$radius;
            margin-bottom:10px;

            .blockchain {
                font-size: $small;
                padding:4px 8px;
                margin-right:10px;
                background:$blue;
                color:$white;
                display:inline-block;
            }

            .contract {
                padding-right:20px;
                display:inline-block;
                font-size: $medium;
                font-weight: bold;
            }

            .actions {
                margin-top:20px;
                display:flex;
                flex-direction: column;

                .action {
                    display:flex;
                    justify-content: space-between;
                    align-items: center;
                    padding:10px;
                    border:1px solid $lightgrey;
                    border-radius:$radius;
                    width:100%;
                    margin-bottom:5px;

                    &:last-child {
                        margin-bottom:0;
                    }

                    .action-name {

                    }

                    .remove {
                        margin-left:20px;
                    }
                }
            }


        }
    }

</style>
