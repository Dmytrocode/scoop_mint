<script lang="ts">
  import { onMount } from "svelte";
  import { Connection } from "@solana/web3.js";
  import { Provider, web3 } from "@project-serum/anchor";
  import { fade } from "svelte/transition";
  import Button from "./components/Button.svelte";
  import Header from "./components/CardHeader.svelte";

  import { candyMachineState, userState } from "./lib/store";
  import {
    getCandyMachineState,
    checkWalletConnected,
    getUserBalance,
    existsOwnerSPLToken,
  } from "./lib/state-helpers";

  /***********************************/
  // Customise the app by changing the following variables. TEST
  const TITLE = "SolScoops (ROUND 1)";
  const DESCRTIPTION = "Mint 1 of the 800 Unique SolScoops!";
  let HEADER_TITLE = "/logo.png";
  
  const HEADER_LINK = "solscoops.com";
  // Your image or GIF needs to be in the /public folder for this to work
  const IMAGE_LINK = "/scoops.gif";
  /***********************************/


  const { solana } = window as any;
  const rpcUrl = import.meta.env.VITE_APP_SOLANA_RPC_HOST?.toString();
  const cluster = import.meta.env.VITE_APP_SOLANA_NETWORK?.toString();
  const candyMachineId = import.meta.env.VITE_APP_CANDY_MACHINE_ID?.toString();
  const opts = { preflightCommitment: "processed" };

  let siteLoading = true;
  let errorOcurred = false;
  let connection: Connection;
  let provider: Provider;
  let candyMachinePublicKey: web3.PublicKey;

  $: itemsRedeemed = $candyMachineState?.state.itemsRedeemed;
  $: itemsAvailable = $candyMachineState?.state.itemsAvailable;

  function checkEnvironmentVariables() {
    // Check if populated
    if (!rpcUrl || !candyMachineId || !cluster) {
      if (!rpcUrl) {
        console.error("RPC URL not populated");
      }
      if (!candyMachineId) {
        console.error("Candy Machine ID not populated");
      }
      if (!cluster) {
        console.error("Environment not populated");
      }
      return true;
    }
    if (candyMachineId.length < 32 || candyMachineId.length > 44) {
      console.error(
        "Candy Machine Public Key is invalid. Enter a length in-between 32 and 44 characters"
      );
      return true;
    }
    return false;
  }

  onMount(async () => {
    // Check if environement variables are populated
    errorOcurred = checkEnvironmentVariables();
    if (errorOcurred) {
      return;
    }

    // If env variables populated, create provider, PK and connection
    connection = new Connection(rpcUrl);
    provider = new Provider(
      connection,
      solana,
      opts.preflightCommitment as web3.ConfirmOptions
    );
    candyMachinePublicKey = new web3.PublicKey(candyMachineId);

    // Get candy machine state
    $candyMachineState = await getCandyMachineState(
      candyMachinePublicKey,
      provider
    );

    // Establish connection to wallet
    if (solana?.isPhantom) {
      $userState.walletPublicKey = await checkWalletConnected(solana);
      if ($userState.walletPublicKey) {
        // Get User Balance
        $userState.userBalance = await getUserBalance(
          $userState.walletPublicKey,
          connection
        );
        // Check if user is whitelisted (ie. check if they have token)
        $userState.isWhiteListed = await existsOwnerSPLToken(
          $userState.walletPublicKey,
          connection,
          $candyMachineState.state.whitelistMintSettings?.mint
        );
      }
    }

    // Stop loading
    siteLoading = false;
  });
</script>

<main class="h-screen">
  
  <!-- Error section -->
  {#if errorOcurred}
    <div class=" h-full flex">
     
      <div class="m-auto">
        An error occurred. Please check if your environment variables have been
        populated correctly and redeploy the applcation.
      </div>
    </div>
  
    <!-- Loading Section -->
  {:else if siteLoading && !errorOcurred}
    <div class=" h-full flex">
      <div class="lds-hourglass m-auto" />
    </div>
  {:else}


    
    <!-- Menu Bar -->
    {#if HEADER_TITLE}
    <div class="head">
      <img src={HEADER_TITLE} alt="logo" class="logoheader"/>
    </div>
    {/if}
    <!-- Card -->
    
    <div
      class="max-w-lg mx-auto rounded-lg my-12  border-2"
      transition:fade style="background-color: BlanchedAlmond; border-radius: 15px">

      <!-- Top Bar -->
      <Header />
      <hr />
      <br />
      <!-- Main Body -->
      <div class="p-6">
        <img src={IMAGE_LINK} alt="" class=" w-1/2 mx-auto m-2" style="border-radius: 15px;"/>
        <div
          class=" text-lg sm:text-2xl font-mono font-bold py-5 tracking-wider"
        >
          {TITLE}
        </div>
        <div class="text-sm sm:text-md font-semibold pb-5 text-blue-600 ">
          {DESCRTIPTION}
        </div>
        
        <div class="time">
          <p style="color: white;">January 16  |  20:00 UTC</p>
        </div>
        
        <div class=" tracking-widest font-bold text-sm pt-3 text-gray-400">
          <!-- {itemsRedeemed} <div class="time">
          <p style="color: white;">January 16  |  20:00 UTC</p>
        </div>-->

          <p style="color: black;">15/{itemsAvailable} claimed </p>
        </div>
        <div class="flex flex-col pt-3">
          {#if $userState.solanaExplorerLink}
            <a
              href={$userState.solanaExplorerLink}
              target="_blank"
              class="text-purple-700 font-semibold  p-1"
              >View on Solana Explorer</a
            >
          {/if}
        </div>
      </div>
      
</div>

  <div class="social">

      <a href="https://twitter.com/SolScoops_NFT">
        <img src="/twitter.png" class="twitter" alt="twit">
    </a>

    <a href="https://discord.gg/VbqJVkVh25">
        <img src="discord.png" class="discord" alt=disc>
        
    </a>
  </div>

  {/if}
</main>
