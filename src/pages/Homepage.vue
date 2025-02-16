<template>
    <div class="bod">
        <div class="centered">
            <h1 class="centeredText" v-if="text">{{ text }}</h1>
            <h1 class="centeredText" v-else>Fruity Countries</h1>
            <b-row class="searchbox">
                <b-form-input class="centeredText" v-model="text" v-on:keyup.enter="searchCountry()" placeholder="Please enter the name of a country"></b-form-input>
            </b-row>
            <b-row cols="2">
                <b-button variant="outline-primary" @click="searchCountry()">Search</b-button>
                <b-button variant="outline-danger" @click="feelingLucky()">I'm feeling lucky</b-button>
            </b-row>
            <small>You can also type the name of a region, subregion, or currency!</small>
        </div>
        <b-container fluid>
            <b-row cols-sm="1" cols-md="2" cols-lg="3" cols-xl="4">
                <Country 
                    v-for="country in countries"
                    :key="country.ccn3"
                    :country="country" 
                />
            </b-row>
        </b-container>
    </div>
</template>

<script>
    import axios from 'axios'
    import Country from '@/components/Country'

    export default {
        name: 'Homepage',
        components: {
            Country
        },
        data() {
            return {
                text: '',
                countries: []
            }
        },
        methods: {
            async searchCountry() {
                // Displaying error message if search term was empty
                if(!this.text) {
                    alert("Please enter a search term!")
                    return
                }
                // Defining requests as constants
                const firstRequest = axios.get(`https://restcountries.com/v3.1/name/${this.text}`)
                const secondRequest = axios.get(`https://restcountries.com/v3.1/subregion/${this.text}`)
                const thirdRequest = axios.get(`https://restcountries.com/v3.1/currency/${this.text}`)
                // Using Promise.allSettled instead of Promise.all so that some requests can fail
                Promise.allSettled([firstRequest,secondRequest,thirdRequest])
                // Using axios.spread to "spread" the requests out into an array
                .then(axios.spread((...responses) => {
                    // Defining responses as constants..
                    // Ordering responses from least to most broad so that "EUR" will definitely return the currency,
                    // rather than "Europe", for example.
                    const thirdResponse= responses[0]
                    const secondResponse = responses[1]
                    const firstResponse = responses[2]
                    // Logging each response
                    console.log("First Response: ",firstResponse)
                    console.log("Second Response: ",secondResponse)
                    console.log("Third Response: ",thirdResponse)
                    // Looping through the responses in the array 
                    responses.forEach(response => {
                        // Checking to see if the status of a response came back as fulfilled
                        if(response.status == "fulfilled"){
                            // If it was, log it and assign this.countries the value of its data array
                            console.log("Fulfilled Response: ", response.value.data)
                            this.countries = response.value.data
                        }
                    });
                    // If all responses return with a status of "rejected", display an alert message and clear the search box
                    if(responses[0].status == "rejected" && responses[1].status == "rejected" && responses[2].status == "rejected"){
                        alert("No results... Try again!")
                        this.text = ""
                    }
                }))
                .catch(errors => {
                    console.log('Errors detected: ', errors)
                })
            },
            feelingLucky(){
                if(!this.text) {
                    alert("Please enter a search term!")
                    return
                }
                axios
                    .get(`https://restcountries.com/v3.1/name/${this.text}`)
                    .then((response) => {
                        console.log("Feeling lucky response: ", response.data)
                        // If the length of the returned array is 1
                        if(response.data.length===1){
                            // Push the name of the country in the response to the URL
                            this.$router.push("/countries/"+this.text)
                        } else {
                            // Otherwise, alert user and clear the search box
                            alert("No result / multiple results found!")
                            this.text = ""
                        }
                    })
                    .catch(error => console.log(error))
            }
        }
    }
</script>

<style scoped>

    h1{
        font-size: 3rem;
        font-family: "Permanent Marker";
        margin: 0.2rem 0 0.7em 0;
        animation: slow-rotate 4s infinite;
    }

    small{
        font-family:'Lucida Sans', 'Lucida Sans Regular', 'Lucida Grande', 'Lucida Sans Unicode', Geneva, Verdana, sans-serif;
    }

    .bod{
        height: 75vh;
        display: grid;
    }

    .centered{
        margin: auto;
    }

    .centeredText{
        text-align: center;
    }
    
    .searchbox{
        margin-bottom: 0.5em;
    }

	@keyframes slow-rotate {
        0%{
            transform: rotate(2deg);
        }
        50%{
            transform: rotate(-2deg);
        }
        100%{
            transform: rotate(2deg);
        }
    }

</style>