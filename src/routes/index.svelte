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
                console.log('Total wave count... ', count.toNumber());

            } else {
                console.log("Ethereum object doesn't exist!");
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

        <button class="waveButton" on:click={null}>
            ⭐ Star
        </button>

        {#if count}
            <p>
                I have {count} stars!
            </p>
        {/if}
    </div>

    {#if !currentAccount}
        <button class="waveButton" on:click={connectWallet}>
            Connect Wallet
        </button>
    {/if}
</main>

<style>

</style>