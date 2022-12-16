<script>
  import { ethers } from "ethers";
  import { GelatoRelaySDK } from "@gelatonetwork/relay-sdk";
  import detectEthereumProvider from "@metamask/detect-provider";
  import { onMount } from "svelte";
  import Web3 from "web3";

  // Replace this with your own API key!
  const SPONSOR_API_KEY = "L_RMvxJ_LhGpctJ9lHWxaSm9iBu98_vPt5jOvPnWTT4_";

  let count = 0;

  onMount(async () => {
    let provider = await detectEthereumProvider();
    if (provider) {
      console.log("MetaMask is installed!");
    } else {
      console.log("MetaMask is not installed!");
    }
    ethereum
      .request({ method: "eth_requestAccounts" })
      .then((_accounts) => console.log("accounts: ", _accounts))
      .catch((err) => {
        console.error(err);
      });
  });

  const increment = async () => {
    // Set up on-chain variables, such as target address
    const counter = "0x30d97B13e29B0cd42e6ebd48dbD9063465bF1997";
    const abi = ["function incrementContext()"];
    const provider = new ethers.providers.Web3Provider(window.ethereum);
    const signer = provider.getSigner();
    const user = await signer.getAddress();
    console.log(signer);

    const signature = await signer.signMessage("test Massage");

    // Generate the target payload
    const contract = new ethers.Contract(counter, abi, signer);
    const { data } = await contract.populateTransaction.incrementContext();

    // Populate a relay request
    const request = {
      chainId: Number(await ethereum.request({ method: "eth_chainId" })),
      target: counter,
      data: data,
      user: user,
    };
    console.log(user);

    // Without a specific API key, the relay request will fail!
    // Reach out to us on Discord to discuss getting an API key.
    // Send a relay request using GelatoRelaySDK!
    const relayResponse = await GelatoRelaySDK.relayWithSponsoredUserAuthCall(
      request,
      provider,
      SPONSOR_API_KEY
    );

    console.log(relayResponse);
    
  };
</script>

<button on:click={increment}>
  send
</button>
