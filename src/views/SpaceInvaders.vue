<template>
    <section class="space-invaders">

        <h1>Let's play some Space Invaders!</h1>
        <hr>
        <p>
            We won't be recording every movement into the blockchain. What we <b>will</b> be storing is a list of high scores.
        </p>


        <hr>

        <section class="panel">

            <section class="box" v-if="!identity">
                <figure class="header">
                    <h1>
                        Oh you want to play Space Invaders?
                    </h1>
                </figure>
                <p>
                    In order to save your high scores we're going to need your Identity.
                </p>

                <button v-on:click="requestIdentity">Provide Identity</button>
            </section>

            <section class="box" v-else>
                <figure class="header">
                    <h1>
                        Lock and load!
                    </h1>
                </figure>
            </section>


            <section class="info">
                <h1>Your Identity name is your Player name.</h1>
                <br>
                <p>
                    Account names are subject to change since you can swap out your account inside of your Identity. We don't want that to
                    happen in this case because your high score would get lost if you changed accounts!
                    <br><br>
                    Because of that Scatter provides the game with the Identity name of your choosing, and an account to sign the transaction of
                    recording your high score with. Woohoo, now you're free to change accounts and keep your high score.
                </p>
            </section>
        </section>

        <hr>

        <section v-if="identity" class="panel">
            <div id="phaser-example"></div>
        </section>

        <section v-else class="panel">
            <h1><i class="fa fa-exclamation-triangle"></i> Provide an Identity to start playing</h1>
        </section>

        <hr>

        <section class="panel">

            <section class="box">
                <figure class="header">
                    <h1>
                        Last High Score<br>
                        <b>{{lastHighScore}}</b>
                    </h1>
                </figure>

            </section>


            <section class="info">
                <h1>High Scores.</h1>
                <br>
                <figure v-for="highScore in highScores">
                    <b>{{highScore.score}}</b> - {{highScore.identity}}
                </figure>
            </section>
        </section>



    </section>
</template>

<script>
    import { mapActions, mapGetters, mapState } from 'vuex'
    import * as Actions from '../store/constants';
    import {RouteNames} from '../vue/Routing'
    import * as SpaceInvaders from '../games/SpaceInvaders'

    import Eos from 'eosjs';

    export default {
        data(){ return {
            lastHighScore:0,
            localEos:null,
            highScores:[]
        }},
        computed: {
            ...mapState([
                'scatter',
                'eos',
                'identity',
            ])
        },
        mounted(){
            setTimeout(() => {
                this.getHighScores();
            }, 500);
        },
        methods: {
            requestIdentity(){
                this.scatter.getIdentity(['account']).then(id => {
                    if(!id) return false;
                    this.scatter.useIdentity(id.hash);
                    this[Actions.SET_IDENTITY](id);
                    SpaceInvaders.load(this.gameOver)
                }).catch(e => console.log(e))
            },
            gameOver(score){
                if(score > this.lastHighScore) {
                    this.lastHighScore = score;
                    const options = {
                        scope: ['invaders', this.identity.account.name],
                        authorization: [`${this.identity.account.name}@active`]
                    };
                    this.eos.contract('invaders').then(contract => {
                        contract.score(this.identity.account.name, score, this.identity.account.name, options)
                    });
                }
            },
            getHighScores(){
                this.eos.getTableRows({
                    "json": true,
                    "scope": 'invaders',
                    "code": 'invaders',
                    "table": "scores",
                    "limit": 500
                }).then(result => {
                    this.highScores = result.rows.sort((a,b) => b.score - a.score);
                    setTimeout(() => {
                        this.getHighScores();
                    }, 2000);
                })
            },
            ...mapActions([
                Actions.SET_IDENTITY
            ])
        }
    }
</script>

<style lang="scss">

</style>