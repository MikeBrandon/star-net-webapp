<script lang="ts">
import { onMount } from "svelte";
import { ethers } from "ethers";
import contractABI from '$lib/utils/StarNet.json';

    let currentAccount;
    let contractAddress = '0x9a70FBfDb4297e5e4f326adBa7D5D95FF5C02DFe';
    let count;

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
        } catch (error) {
            console.log(error);
        }
    }

    async function getStars() {
        try {
            const { ethereum } = window;

            if (ethereum) {
                const provider = new ethers.providers.Web3Provider(ethereum);
                const signer = provider.getSigner();
                const starSmartContract = new ethers.Contract(contractAddress, contractABI.abi, signer);

                count = await starSmartContract.getTotalStars();
                console.log('Total star count... ', count.toNumber());

            } else {
                console.log("Ethereum object doesn't exist!");
            }

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

                const starTxn = await starSmartContract.sendStar();
                console.log('Mining...', starTxn.hash);

                await starTxn.wait();
                console.log('Mined --', starTxn.hash);

                count = await starSmartContract.getTotalStars();
                console.log("Retrieved total star count...", count.toNumber());

                alert('Wow! Thanks for the Star.');
            }

        } catch (error) {
            console.log(error);
        }
    }

    onMount(() => {
        checkIfWalletIsConnected();
        getStars();
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
                <button class="starButton" on:click={sendStar}>
                    ⭐ Star
                </button>
            {/if}
        </div>

        <div>
            {#if count}
                <p>
                    I have {count} ⭐{count == 1 ? 'star!' : 'stars!'}
                </p>
            {/if}
        </div>

    </div>
</main>

<style>
    main {
        background-color: #0d1116;

        display: flex;
        flex-direction: column;
        justify-content: center;
        align-items: center;
        text-align: center;
        height: 100vh;

        font-size: 2rem;
        font-family: -apple-system, Inter, BlinkMacSystemFont, 'Segoe UI', 'Roboto',
		'Oxygen', 'Ubuntu', 'Cantarell', 'Fira Sans', 'Droid Sans', 'Helvetica Neue',
		sans-serif;
        font-weight: bold;
    }

    main > div > div {
        color: white;
        margin-bottom: 1rem;
    }

    button {
        width: 100%;
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
</style>