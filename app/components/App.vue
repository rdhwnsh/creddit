<template>
    <Page>
        <ActionBar :title="apptitle" />
        <FlexboxLayout>
            <PullToRefresh @refresh="refreshList">
                <StackLayout>
                    <!-- SHOWS WHICH PAGE -->
                    <Label class="message"> {{page}} / 9 </Label>
                    <!-- INPUT FOR ENTERING SUBREDDIT -->
                    <TextField v-model="subreddit" hint="Enter a subreddit" @textChange="subredditValidation"
                        returnKeyType="done"></TextField>
                    <!-- NEXT POST AND PREVIOUS POST BUTTONS -->
                    <button @tap="nextpost"> Next </button>
                    <button @tap="previouspost"> Previous </button>
                    <!-- <button @tap="subredditValidation"> RSR </button> -->
                    <!-- <Label class="message bold" textWrap="true">Displaying posts from {{subreddit}} </Label> -->
                    <!-- VIEW SUBREDDIT -->
                    <StackLayout v-if="dataverified">
                        <ScrollView orientation="vertical">
                            <StackLayout orientation="vertical">
                                <Image class="img-rounded" :src="img" />
                                <!-- CALL OPENPOST WHEN THE TITLE IS CLICKED -->
                                <Label class="message bold" textWrap="true" @tap="openpost">{{title}} </Label>
                                <Label class="message" textWrap="true">{{usernameDisplay}} </Label>
                            </StackLayout>
                        </ScrollView>
                    </StackLayout>
                    <StackLayout v-else>
                        <ScrollView orientation="vertical">
                            <StackLayout orientation="vertical">
                                <Label class="message bold" textWrap="true" @tap="openpost">Data Is Loading... </Label>
                            </StackLayout>
                        </ScrollView>
                    </StackLayout>
                </StackLayout>
            </PullToRefresh>
        </FlexboxLayout>
    </Page>
</template>

<script>
    var Toast = require("nativescript-toast")
    import {registerElement} from "nativescript-pulltorefresh";
    var utilityModule = require("utils/utils");

    export default {
        data() {
            return {
                usernameDisplay: "",
                page: 0,
                apptitle: "",
                data: [],
                subreddit: "todayilearned",
                title: "",
                img: "",
                maxpages: "",
                appversion: "v1.5.1r2",
                dataverified: false,
            }
        },
        methods: {

            // GOTO NEXT POST
            nextpost() {

                // PAGE WILL INCREMENT
                this.page++;

                // IF PAGE IS ABOVE 25 OR BELOW 0, PAGE WILL TURN BACK TO PAGE 0 (TO PREVENT APP CRASHING)
                if (this.page >= this.maxpages || this.page <= 0) {
                    this.page = 0;
                }

                // SAVE USERNAME, TITLE AND IMAGE VARIABLE
                this.usernameDisplay = "/u/" + this.data[this.page].data.author
                this.title = this.data[this.page].data.title
                this.img = this.data[this.page].data.thumbnail

                // CONSOLE.LOG THE PAGE NUMBER
                console.log(this.page);
            },

            // GOTO PREVIOUS POST
            previouspost() {

                // PAGE WILL DECREMENT

                // When users press previous button (when at page 0) , they will now instead go this.maxpages instead of staying at page 0 (UPDATE 1.3.0)
                if (this.page >= this.maxpages + 1) {
                    this.page = 0;

                } else if (this.page > 0) {
                    this.page--;

                } else if (this.page <= 0) {
                    this.page = this.maxpages;
                }

                // SAVE USERNAME, TITLE AND IMAGE VARIABLE
                this.usernameDisplay = "/u/" + this.data[this.page].data.author
                this.title = this.data[this.page].data.title
                this.img = this.data[this.page].data.thumbnail

                // CONSOLE.LOG THE PAGE NUMBER
                console.log(this.page);

            },

            // GET SUBREDDIT DATA
            getSub() {

                this.dataverified = false;
                
                fetch("https://www.reddit.com/r/" + this.subreddit + "/new.json?limit=100")
                    .then(response => response.json())
                    .then(json => {
                        // SAVE DATA TO DATA VARIABLE                        
                        this.data = json.data.children

                        // TURNS TO PAGE O
                        this.page = 0;

                        // SAVE THE DATA TO VARIABLE
                        this.maxpages = this.data.length - 1;
                        this.usernameDisplay = "/u/" + this.data[this.page].data.author
                        this.title = this.data[this.page].data.title
                        this.img = this.data[this.page].data.thumbnail
                        this.dataverified = true;
                    })
            },

            // REFRESH LIST
            refreshList(args) {

                this.getSub()
                Toast.makeText("Refreshing...").show()

                var pullRefresh = args.object;
                setTimeout(function () {
                    pullRefresh.refreshing = false;
                }, 1000);
            },

            // OPENS POST IN BROWSER
            openpost() {

                // GET THE PERMALINK BY USING THE PAGE NUMBER (INDEX)
                let permalink = this.data[this.page].data.permalink

                // FULL URL
                let url = "https://www.reddit.com" + permalink
                utilityModule.openUrl(url);
            },

            getRandomSub(){

                    this.dataverified = false;

                    // GETTING DATA LIMIT IS 100 ALWAYS
                    let limit = 100;

                    Toast.makeText("Getting all the subreddits...").show();

                    fetch("https://www.reddit.com/reddits.json?limit=" + limit)
                        .then(response => response.json())
                        .then(json => {
                            this.subreddit = json.data.children[Math.floor(Math.random() * limit)].data.display_name;
                            console.log(this.subreddit)
                            Toast.makeText("You got a random subreddit: " + this.subreddit).show();
                            this.dataverified = true;
                        })
            },
            
            // GETS RANDOM SUBREDDIT
            subredditValidation() {
                let keyword = "getrandomsub"

                if (this.subreddit.toLowerCase() == keyword) {
                    this.getRandomSub()
                } else {
                    this.getSub()
                }


            }
        },

        created() {
            this.apptitle = "☝️ creddit " + this.appversion
        },

        mounted() {
            var welcometoast = Toast.makeText("Welcome!").show();
            this.getSub();
        }

    }
</script>

<style scoped>
    FlexboxLayout {
        horizontal-align: center;
        vertical-align: center;

    }

    StackLayout {
        width: 90%;
        height: 95%;
    }

    ActionBar {
        background-color: #ffcdd2;
        color: #000000;
    }

    .message {
        /* text-align: center; */
        font-size: 20;
        color: #333333;
    }

    button {
        background-color: #b2dfdb;
        color: #000;
    }

    Image {
        horizontal-align: left;
        text-align: left;
        vertical-align: left;
        width: 100;
        border-radius: 100;
    }

    .bold {
        font-weight: bold;
    }
</style>