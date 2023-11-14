<script setup lang="ts">
import { Trade } from "../vuex/api/Trade";
import { Header as HeaderManager } from "../vuex/api/Header";
import { computed, onUnmounted } from "@vue/runtime-core";
import { store } from "/@/vuex/store";
import { onMounted, ref } from "vue";
import { useI18n } from "vue-i18n";
import { ACTIONS } from "/@/vuex/modules/enums/StoreEnums";
import {
    createWeb3Modal,
    defaultWagmiConfig,
    useWeb3Modal,
    useWeb3ModalState,
    useWeb3ModalTheme,
} from "@web3modal/wagmi/vue";
import { mainnet, arbitrum, polygon } from "@wagmi/core/chains";
import eventBus from "/@/api/EventBus";


const { t } = useI18n();
let TradeManager;

let usdcSizeSaved: string = "";
let optionSizeSaved: string = "";


const selectedFormState = computed(() => store.getters.selectedFormState).value;
const web3Connected = computed(() => store.getters.getWeb3Connected).value;
let userAcc: string | null = sessionStorage.getItem("account");
const quoteRequest = computed(() => store.getters.quoteRequest).value;
const legsCurrent = computed(() => store.getters.legsCurrent).value;
const tradeForm = computed(() => store.getters.getTradeForm).value;
const chainId = computed(() => store.getters.getChainId).value;
let connectTogglerLabel = ref(t("connectWallet"));
let isFirstLoad = false;
let connected = ref(false);

let modal;

onMounted(() => {
    isFirstLoad = false;
    TradeManager = new Trade();

    const projectId = import.meta.env.VITE_VUE_PROJECT_ID;

    const metadata = {
        name: 'Web3Modal',
        description: 'Web3Modal Connect',
        cacheProvider: false,
        url: 'https://web3modal.com',
        icons: ['https://avatars.githubusercontent.com/u/37784886']
    }
    //
    const chains = [mainnet, arbitrum, polygon]
    // const chains = [];
    const wagmiConfig = defaultWagmiConfig({ chains, projectId, metadata });
    //
    createWeb3Modal({ wagmiConfig, projectId, chains });
    const { setThemeMode, setThemeVariables, themeMode, themeVariables } = useWeb3ModalTheme();
    //
    //
    setThemeMode('dark');
    //
    setThemeVariables({
        '--w3m-color-mix': '#6030f8',
        '--w3m-color-mix-strength': 40
    });

    modal = useWeb3Modal();
    // // const { open, close } = useWeb3Modal();
    // modal.close();

    eventBus.on('userAccount', (userAccount: string | null) => {
        userAcc = userAccount;
        connectToggler();
        console.log('userAccount', userAccount);
    });

    userAcc = sessionStorage.getItem("account");

    if (userAcc) {
        const manager = new HeaderManager();
        if (manager.minifyAddress(userAcc).startsWith("unde")) {
            connectTogglerLabel.value = t("connectWallet");
            console.log('UNDEFINED userAcc', userAcc);
            return t("connectWallet");
        }
        console.log('userAcc', userAcc);
        connectTogglerLabel.value = t("account") + manager.minifyAddress(userAcc);
        return t("account") + manager.minifyAddress(userAcc);
    }
});

onUnmounted(() => {
    isFirstLoad = false;
    console.log('onUnmounted', );
    // sessionStorage.removeItem("account");
});

const connectToggler = async () => {
    const manager = new HeaderManager();
    const accounts = await (window as any).ethereum.request({method: 'eth_accounts'});
    const account = accounts[0];
    sessionStorage.setItem("account", account);

    store.dispatch(ACTIONS.USER_ACCOUNT, account);
    if (!account && !userAcc) {
        connectTogglerLabel.value = t("connectWallet");
        return t("connectWallet");
    } else {
        connectTogglerLabel.value = t("account") + manager.minifyAddress(account);
        return t("account") + manager.minifyAddress(account);
    }
};
// const connected = () => {
//     return this.web3Connected ? "button__navigation--connect" : "";
// };

let usdcSize = tradeForm.usdcSize;

let optionSize = tradeForm.optionSize;

const connect = async () => {
    connected = ref(true);
    isFirstLoad = true;
    eventBus.emit("connectButtonConnect", userAcc);
    usdcSizeSaved = tradeForm.usdcSize;
    optionSizeSaved = tradeForm.optionSize;
    const selectedNetwork = computed(() => store.getters.selectedNetwork).value || localStorage.getItem("selectedNetwork");
    const chainID = computed(() => store.getters.getChainId).value;
    await store.dispatch(ACTIONS.CONNECT_USER, chainID);

    // const projectId = import.meta.env.VITE_VUE_PROJECT_ID;
    //
    // const metadata = {
    //     name: 'Web3Modal',
    //     description: 'Web3Modal Connect',
    //     url: 'https://web3modal.com',
    //     icons: ['https://avatars.githubusercontent.com/u/37784886'],
    // }
    //
    // const chains = [mainnet, arbitrum, polygon];
    // const wagmiConfig = defaultWagmiConfig({ chains, projectId, metadata });
    // createWeb3Modal({ wagmiConfig, projectId, chains });
    //
    //
    // const { setThemeMode, setThemeVariables, themeMode, themeVariables } = useWeb3ModalTheme();
    //
    // setThemeMode('dark');
    //
    // modal = useWeb3Modal();
    // // open({ view: 'Account' })
    // // await modal.open();

    const projectId = import.meta.env.VITE_VUE_PROJECT_ID;

    const metadata = {
        name: 'Web3Modal',
        description: 'Web3Modal Connect',
        cacheProvider: false,
        url: 'https://web3modal.com',
        icons: ['https://avatars.githubusercontent.com/u/37784886']
    }
    //
    const chains = [mainnet, arbitrum, polygon]
    // const chains = [];
    const wagmiConfig = defaultWagmiConfig({ chains, projectId, metadata });
    //
    createWeb3Modal({ wagmiConfig, projectId, chains });
    const { setThemeMode, setThemeVariables, themeMode, themeVariables } = useWeb3ModalTheme();
    //
    //
    setThemeMode('dark');
    //
    setThemeVariables({
        '--w3m-color-mix': '#6030f8',
        '--w3m-color-mix-strength': 40
    });

    modal = useWeb3Modal();
    // const { open, close } = useWeb3Modal();
    // modal.close();


    store.dispatch(ACTIONS.CHANGE_CHAIN, selectedNetwork);
    store.dispatch(ACTIONS.CHANGE_NETWORK, selectedNetwork);
    console.log('connect CONNECT BUTTON CHANGE CHAIN selectedNetwork', selectedNetwork);

    await connectToggler();
    // const { open, selectedNetworkId } = useWeb3ModalState();
    //
    // if (!open) {
    //     await connectToggler();
    // }m

    optionSize = optionSizeSaved;
    usdcSize = usdcSizeSaved;
    // await TradeManager.updateUserBalances();

    userAcc = sessionStorage.getItem("account");
    console.log('userAcc', userAcc);

    if (userAcc) {
        const manager = new HeaderManager();
        if (manager.minifyAddress(userAcc).startsWith("unde")) {
            connectTogglerLabel.value = t("connectWallet");
            return t("connectWallet");
        }
        connectTogglerLabel.value = t("account") + manager.minifyAddress(userAcc);
        return t("account") + manager.minifyAddress(userAcc);
    }

    // optionSize = optionSizeSaved;
    // usdcSize = usdcSizeSaved;
    // // await TradeManager.updateUserBalances();
    //
    // store.dispatch(ACTIONS.CHANGE_CHAIN, selectedNetwork);
    // store.dispatch(ACTIONS.CHANGE_NETWORK, selectedNetwork);
    // console.log('connect CONNECT BUTTON CHANGE CHAIN selectedNetwork', selectedNetwork);
    connected = ref(false);
    window.location.reload();
};
</script>

<template>
<!--    <div class="connect-btn">-->
        <w3m-connect-button
            :label="connectTogglerLabel"
            @click="connect"
        />
<!--    </div>-->

</template>


<style scoped>
    /*w3m-connect-button {*/
    /*   margin-right: 10px;*/
    /*}*/

    .connect-btn {
        background: linear-gradient(262.01deg, #6030f8 -4.3%, #451ec1 97.68%);
        border: 1px solid #451ec1;
        border-radius: 50px;
        display: flex;
        padding: 10px 20px;
        color: #FFFFFF;
        justify-content: center;
        align-items: center;
        font-weight: 800;
        cursor: pointer;
    }
</style>
