<script lang="ts">
import { onMount } from "svelte";
import { ethers } from "ethers";
import contractABI from '$lib/utils/StarNet.json';
import { Jumper } from 'svelte-loading-spinners';
import confetti from 'canvas-confetti';

    let currentAccount;
    let contractAddress = '0x4C92BEBfa0b92d2B6B3EEEB5d6743839F10CF3Cc';
    let count;
    let allStars;
    let messageText = '';
    let loading, isWinner = false;
    let someoneJustWon = [false, null];
    let safeTime: any = 0;
    let update = 0;
    
    var colors = ["#60c657", "#35aee2"];

    function castConfetti() {
        confetti({
            particleCount: 2,
            angle: 60,
            spread: 55,
            origin: { x: 0 },
            colors: colors,
        });
        confetti({
            particleCount: 2,
            angle: 120,
            spread: 55,
            origin: { x: 1 },
            colors: colors,
        });

        if (Date.now() < Date.now() + 15000) {
            requestAnimationFrame(castConfetti);
        }
    }

    async function getAllStars() {
        try {

            const { ethereum } = window;

            if (ethereum) {
                const provider = new ethers.providers.Web3Provider(ethereum);
                const signer = await provider.getSigner();
                const starSmartContract = new ethers.Contract(contractAddress, contractABI.abi, signer);

                const lastStarred = await starSmartContract.getLastStar();
                const lastStar = new Date(lastStarred * 1000);
                safeTime = new Date(lastStar.getTime() + 15*60000);

                console.log('Last Star: ', lastStar);

                count = await starSmartContract.getTotalStars();
                console.log('Total star count... ', count.toNumber());

                const stars = await starSmartContract.getAllStars();
                console.log(stars);

                let starsCleaned = [];
                stars.forEach(star => {
                    starsCleaned.push({
                        address: star.sender,
                        timestamp: new Date(star.timestamp * 1000),
                        message: star.message,
                        won: star.won
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

    async function setupNewStarEventListener() {
        try {
            
            const { ethereum } = window;

            if (ethereum) {
                const provider = new ethers.providers.Web3Provider(ethereum);
                const signer = provider.getSigner();
                const starSmartContract = new ethers.Contract(contractAddress, contractABI.abi, signer);

                starSmartContract.on('NewStar', async (from, timestamp, message) => {
                    console.log('from', from);
                    const stars = await starSmartContract.getAllStars();
                    console.log(stars);

                    let starsCleaned = [];
                    stars.forEach(star => {
                        starsCleaned.push({
                            address: star.sender,
                            timestamp: new Date(star.timestamp * 1000),
                            message: star.message,
                            won: star.won
                        });
                    });

                    allStars = [...starsCleaned];   
                    console.log(allStars);
                })

                starSmartContract.on('NewWinner', (address) => {
                    if (currentAccount.trim().toLowerCase() == address.trim().toLowerCase()) {
                        isWinner = true;
                        castConfetti();
                    } else {
                        someoneJustWon = [true, address];
                    }
                })
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
                console.log(`*${currentAccount}*`);
                getAllStars();
                setupNewStarEventListener();
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
            setupNewStarEventListener();
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

                const starTxn = await starSmartContract.sendStar(messageText, { gasLimit: 300000 });
                console.log('Mining...', starTxn.hash);

                loading = true;
                await starTxn.wait();
                console.log('Mined --', starTxn.hash);

                const lastStarred = await starSmartContract.getLastStar();
                const lastStar = new Date(lastStarred * 1000);
                safeTime = new Date(lastStar.getTime() + 15*60000);

                count = await starSmartContract.getTotalStars();
                console.log("Retrieved total star count...", count.toNumber());
                loading = false;
                update++;
                messageText = '';
            }

        } catch (error) {
            console.log(error);
            messageText = '';
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

    function millisToMinutesAndSeconds(millis) {
        var minutes = Math.floor(millis / 60000);
        var seconds = ((millis % 60000) / 1000).toFixed(0);
        return minutes + "min " + (seconds < 10 ? '0' : '') + seconds + 'sec';
    }

    onMount(() => {
        checkIfWalletIsConnected();
    })
</script>

<main class="mainContainer">
    <div class="dataContainer">
        {#if !isWinner}
            <div class="header">
                👋 Hey there!
            </div>
            {#if someoneJustWon[0]}
                <div class="white">
                    {someoneJustWon[1]} just Won 0.0001ETH!
                </div>
            {/if}
        {:else}
            <div class="large-trophy">
                🏆
            </div>
            <div class="header">
                YOU ARE A WINNER!!!
            </div>
        {/if}

        <div class="bio">
            Leave a message to drop a ⭐Star for a chance to win 0.0001 ETH.
        </div>

        <div>
            {#if !currentAccount}
                <button class="starButton" on:click={connectWallet}>
                    Connect Wallet
                </button>
            {:else}
                {#if !loading}
                    {#key update}
                        {#if safeTime < new Date()}
                            <input class='input' type="text" bind:value={messageText}>
                            <button class="starButton" class:cupo={(messageText.length > 0)} on:click={sendStar} disabled={!(messageText.length > 0)}>
                                Send ⭐Star
                            </button>
                        {:else}
                            <div>
                                You Still have to wait for {millisToMinutesAndSeconds(safeTime.getTime() - new Date().getTime())}.
                            </div>
                        {/if}
                    {/key}
                {/if}
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
                    <div class='star-container' class:won={star.won}>
                        {#if star.won}
                            <div class='trophy'>
                                🏆
                            </div>
                        {/if}
                        <div class='data-holder'>
                            <div class='message'>⭐ {star.message}</div>
                            <div class='small-text'>
                                <div class='address'>
                                    👤<a href="https://rinkeby.etherscan.io/address/{star.address}"> {star.address}</a>
                                </div>
                                <div class='time'>🕒 {getShortDate(star.timestamp)}</div>
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
        height: 80vh;

        font-size: 1.5rem;
        font-family: -apple-system, Inter, BlinkMacSystemFont, 'Segoe UI', 'Roboto',
		'Oxygen', 'Ubuntu', 'Cantarell', 'Fira Sans', 'Droid Sans', 'Helvetica Neue',
		sans-serif;
        font-weight: bold;

        overflow-y: hidden;
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
    }

    .cupo {
        cursor: pointer;
    }

    p {
        background: -webkit-linear-gradient(left, #60c657 30%, #35aee2 60%);
        background-clip: text;
        -webkit-background-clip: text;
        -webkit-text-fill-color: transparent;
    }

    .star-container { 
        background-color: lightgreen;
        margin-top: 0.25rem;
        padding: 1rem;

        color: black;
        text-align: left;

        width: 90%;
        border-radius: 9px;
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

    .small-text {
        display: flex;
        font-size: 0.9rem;
        justify-content: space-between;
        width: 100%;
    }

    .message {
        font-size: 1.2rem;
        margin-bottom: 0.25rem;
    }

    .stars-holder{
        width: 45vw;
        height: 50vh;
        overflow-y: scroll;
        -ms-overflow-style: none;
        scrollbar-width: none;
    }

    .stars-holder::-webkit-scrollbar {
            display: none;
        }

    .data-holder {
        width: 100%;
    }

    .spinner {
        display: flex;
        justify-content: center;
        align-items: center;
    }

    .time {
        text-align: right;
    }

    .won {
        background-color: goldenrod;
        display: flex;
        justify-content: center;
        align-items: center;
    }

    .trophy {
        transform: translateX(-5px)
    }

    .dataContainer {
        display: flex;
        flex-direction: column;
        justify-content: center;
        align-items: center;
    }

    .large-trophy {
        font-size: 10rem;
    }

    .white {
        color: white;
        text-align: center;
        width: 100%;
        font-size: 0.9rem;
    }

    @media only screen and (max-width: 640px) {
        main {
            padding-top: 3rem;
            height: 100vh;
            width: 90vw;
            font-size: 1rem;
            padding: 0.5rem;
        }

        button {
            width: 50%;
            height: 2rem;
            font-size: 1rem;
        }

        .star-container {
            padding: 0.5rem;
            border-radius: 6px;
        }

        .input {
            font-size: 1rem;
            padding-top: 0.25rem;
            padding-bottom: 0.25rem;
            margin-bottom: 1rem;
        }

        .small-text {
            flex-direction: column;
            font-size: 0.5rem;
            width: 100%;
        }

        .message {
            font-size: 0.75rem;
            font-weight: normal;
        }

        /* LOL */

        .stars-holder{
            width: 80%;
            overflow-y: scroll;
        }

        .data-holder {
            width: 100%;
        }

        .time {
            text-align: left;
            margin-top: 0.25em;
        }

        .won {
            background-color: goldenrod;
            display: flex;
            justify-content: center;
            align-items: center;
        }

        .trophy {
            transform: translateX(-2.5px)
        }

        .dataContainer {
            margin-top: 3em;
        }

        .large-trophy {
            font-size: 5rem;
        }
    }
</style>