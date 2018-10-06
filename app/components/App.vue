<template>
    <Page>
        <ActionBar :title="apptitle" />

        <Scrollview>

            <StackLayout>
                <Label class="message"> {{taptimes}} / 25 </Label>
                <TextField v-model="subreddit" hint="Enter a subreddit" @input="getSubreddit"></TextField>
                <button @tap="getSubreddit">Get subreddit list</button>
                <button @tap="nextpost">{{getnextpost}}</button>
                <button @tap="previouspost">{{getpreviouspost}}</button>


                <!-- <Label class="message bold" textWrap="true">Displaying posts from {{subreddit}} </Label> -->


                <Label class="message bold" textWrap="true">{{title}} </Label>
                <Image :src="img" />
                <Label class="message" textWrap="true">{{usernameDisplay}} </Label>

            </StackLayout>

            <!-- <GridLayout colums="*" rows="*">
            <Label class="message" :text="usernameDisplay" col="0" row="0"/>
        </GridLayout> -->


        </Scrollview>
    </Page>
</template>

<script>
    var Toast = require("nativescript-toast")
    export default {
        data() {
            return {
                usernameDisplay: "",
                taptimes: 0,
                apptitle: "dev.bynabil.testingapp",
                data: [],
                subreddit: "todayilearned",
                title: "",
                img: "",
                getnextpost: "Press the button above first!",
                getpreviouspost: "Press the button above first!", 
            }
        },
        methods: {
            nextpost() {
                this.taptimes++;
                this.usernameDisplay = "/u/" + this.data[this.taptimes].data.author
                this.title = this.data[this.taptimes].data.title
                this.img = this.data[this.taptimes].data.thumbnail

                if (this.taptimes >= 24) {
                    this.taptimes = 1;
                }

                console.log(this.taptimes);
            },

            previouspost(){
                this.taptimes--;
                this.usernameDisplay = "/u/" + this.data[this.taptimes].data.author
                this.title = this.data[this.taptimes].data.title
                this.img = this.data[this.taptimes].data.thumbnail

                if (this.taptimes <= 1) {
                    this.taptimes = 1;
                }

                console.log(this.taptimes);
                
            },
            
            getSubreddit() {
                fetch("https://www.reddit.com/r/" + this.subreddit + ".json")
                    .then(response => response.json())
                    .then(json => {
                        this.data = json.data.children
                        this.getnextpost = "Get next post"
                        this.getpreviouspost = "get previous post"

                        this.taptimes++;
                        this.usernameDisplay = "/u/" + this.data[this.taptimes].data.author
                        this.title = this.data[this.taptimes].data.title
                        this.img = this.data[this.taptimes].data.thumbnail

                        var doneToast = Toast.makeText("Done getting subreddit").show();
                    })
            }
        },

        created() {
            var welcometoast = Toast.makeText("Welcome!").show();
            fetch("https://www.reddit.com/r/" + this.subreddit + ".json")
                .then(response => {
                    response.json()
                })
                .then(json => {
                    this.data = json.data.children
                    this.getnextpost = "Get next post"
                    this.getpreviouspost = "get previous post"

                    this.taptimes++;
                    this.usernameDisplay = "/u/" + this.data[this.taptimes].data.author
                    this.title = this.data[this.taptimes].data.title
                    this.img = this.data[this.taptimes].data.thumbnail
                })
        }

    }
</script>

<style scoped>
    ActionBar {
        background-color: #53ba82;
        color: #ffffff;
    }

    .message {
        /* text-align: center; */
        font-size: 20;
        color: #333333;
    }

    button {
        background-color: #53ba82;
        color: #ffffff;
    }

    Image {
        width: 200;
        align-items: left;
        text-align: left;
    }

    .bold {
        font-weight: bold;
    }
</style>