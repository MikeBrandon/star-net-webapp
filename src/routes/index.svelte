<script lang="ts">
import { onMount } from "svelte";
import { ethers } from "ethers";
import contractABI from '$lib/utils/StarNet.json';
import { Jumper } from 'svelte-loading-spinners';

    let currentAccount;
    let contractAddress = '0xe85749ac33738fD1E7430763731270B74b17b52A';
    let count;
    let allStars;
    let messageText = '';
    let loading = false;

    async function getAllStars() {
        try {

            const { ethereum } = window;

            if (ethereum) {
                const provider = new ethers.providers.Web3Provider(ethereum);
                const signer = await provider.getSigner();
                const starSmartContract = new ethers.Contract(contractAddress, contractABI.abi, signer);

                count = await starSmartContract.getTotalStars();
                console.log('Total star count... ', count.toNumber());

                const stars = await starSmartContract.getAllStars();

                let starsCleaned = [];
                stars.forEach(star => {
                    starsCleaned.push({
                        address: star.sender,
                        timestamp: new Date(star.timestamp * 1000),
                        message: star.message
                    });
                });

                allStars = [...starsCleaned];
                console.log(allStars);
            } else {
                console.log('Ethereum Object not Found.')
            }

        } catch (error) {
            console.log(error);
        }
    }

    async function checkIfWalletIsConnected() {
        try {
            const { ethereum } = window;

            if (!ethereum) {
                console.log("Make sure you have metamask!");
                return;
            } else {
                console.log("We have the ethereum object", ethereum);
            }

            const accounts = await ethereum.request({ method: 'eth_accounts' });

            if (accounts.length !== 0) {
                const account = accounts[0];
                console.log('Found and authorized account: ', account);
                currentAccount = account;
                getAllStars();
            } else {
                console.log('No Authorized Account Found!')
            }
        } catch (error) {
            console.log(error);
        }
    }

    async function connectWallet(){
        try {
            const { ethereum } = window;

            if (!ethereum) {
                alert("get Metamask!");
                return;
            }

            const accounts = await ethereum.request({ method: "eth_requestAccounts" });

            console.log('Connected: ', accounts[0]);
            currentAccount = accounts[0];
            getAllStars();
        } catch (error) {
            console.log(error);
        }
    }

    async function sendStar() {
        try {
            
            const { ethereum } = window;

            if (ethereum) {
                const provider = new ethers.providers.Web3Provider(ethereum);
                const signer = provider.getSigner();
                const starSmartContract = new ethers.Contract(contractAddress, contractABI.abi, signer);

                const starTxn = await starSmartContract.sendStar(messageText);
                console.log('Mining...', starTxn.hash);

                loading = true;
                await starTxn.wait();
                console.log('Mined --', starTxn.hash);

                count = await starSmartContract.getTotalStars();
                console.log("Retrieved total star count...", count.toNumber());
                loading = true;

            }

        } catch (error) {
            console.log(error);
        }
    }

    function getShortDate(date: Date) {
        const _date = date.getDate().toString().padStart(2, "0");
        const _month = (date.getMonth() + 1).toString().padStart(2, "0");
        const _year = date.getFullYear();

        const dateString = `${_date}/${_month}/${_year}`;

        const _hour = date.getHours().toString().padStart(2, "0");
        const _minute = date.getMinutes().toString().padStart(2, "0");
        const _seconds = date.getSeconds().toString().padStart(2, "0");

        const timeString = `${_hour}:${_minute}:${_seconds}`;

        return `${dateString} ${timeString}`;
    }

    onMount(() => {
        checkIfWalletIsConnected();
    })
</script>

<main class="mainContainer">
    <div class="dataContainer">
        <div class="header">
        👋 Hey there!
        </div>

        <div class="bio">
        I am Mike. Would you like to drop me a ⭐Star.
        </div>

        <div>
            {#if !currentAccount}
                <button class="starButton" on:click={connectWallet}>
                    Connect Wallet
                </button>
            {:else}
                <input class='input' type="text" bind:value={messageText}>
                <button class="starButton" on:click={sendStar} disabled={!(messageText.length > 0)}>
                    ⭐ Star
                </button>
            {/if}
        </div>

        <div>
            {#if loading}
                <div class = "spinner">
                    <Jumper size="100" color="#35AEE2" unit="px" duration="1s"></Jumper>
                    <p class='loading-text'>Please Wait...</p>
                </div>
            {:else}
                {#if count}
                    <p>
                        I have {count} ⭐{count == 1 ? 'star!' : 'stars!'}
                    </p>
                {/if}
            {/if}
        </div>

        <div class='stars-holder'>
            {#if allStars}
                {#each allStars.reverse() as star}
                    <div class='star-container'>
                        <div class='data-holder'>
                            <div class='message'>{star.message}</div>
                            <div class='small-text'>
                                <div class='address'>
                                    👤<a href="https://rinkeby.etherscan.io/address/{star.address}">{star.address}</a>
                                </div>
                                <div class='time'>🕒{getShortDate(star.timestamp)}</div>
                            </div>
                        </div>
                    </div>
                {/each}
            {/if}
        </div>

    </div>
</main>

<style>
    main {
        background-color: #0d1116;

        padding-top: 10rem;

        display: flex;
        flex-direction: column;
        align-items: center;
        text-align: center;
        height: 100vh;

        font-size: 1.5rem;
        font-family: -apple-system, Inter, BlinkMacSystemFont, 'Segoe UI', 'Roboto',
		'Oxygen', 'Ubuntu', 'Cantarell', 'Fira Sans', 'Droid Sans', 'Helvetica Neue',
		sans-serif;
        font-weight: bold;
    }

    main > div > div {
        color: white;
        margin-bottom: 0.5rem;
    }

    button {
        width: 75%;
        height: 3rem;
        font-size: 1.5rem;
        color: black;
        cursor: pointer;
    }

    p {
        background: -webkit-linear-gradient(left, #60c657 30%, #35aee2 60%);
        background-clip: text;
        -webkit-background-clip: text;
        -webkit-text-fill-color: transparent;
    }

    .star-container { 
        background-color: oldlace;
        margin-top: 0.25rem;
        padding: 1rem;

        color: black;
        text-align: left;

        width: 100%;
    }

    .input {
        background-color: transparent;
        font-size: 1.5rem;
        color: white;
        border: 1px solid white;
        padding-top: 0.5rem;
        padding-bottom: 0.5rem;
        margin-bottom: 1rem;
        border-radius: 6px;
    }

    .num {
        margin-right: 1rem;
    }

    .small-text {
        display: flex;
        font-size: 0.9rem;
        justify-content: space-between;
        width: 100%;
    }

    .message {
        font-size: 1.2rem;
    }

    .stars-holder{
        width: 50vw;
    }

    .data-holder {
        width: 100%;
    }

    .spinner {
        display: flex;
        justify-content: center;
        align-items: center;
    }
</style>