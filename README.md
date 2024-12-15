# Alpha-Social
// File: pages/mint.vue
<script setup lang="ts">
import { ref } from 'vue'
import { useAccount, useConnect, useChainId } from '@alpha/vue'
import { zksyncSepoliaTestnet } from '@alpha/vue/chains'

// State management
const isLoading = ref(false)
const error = ref<string | null>(null)

// Web3 Hooks
const { address, status } = useAccount()
const { connect, connectors } = useConnect()
const chainId = useChainId()

// Mint state
const quantity = ref(1)
const isMinting = ref(false)

// Validate network
const isCorrectNetwork = computed(() => 
  chainId.value === zksyncSepoliaTestnet.id
)

// Switch network handler
const switchNetwork = async () => {
  try {
    await switchChain(zksyncSepoliaTestnet.id)
  } catch (err) {
    error.value = 'Failed to switch network'
    console.error(err)
  }
}

// Mint NFT function
const mintNFT = async () => {
  if (!address.value) {
    error.value = 'Please connect wallet first'
    return
  }

  if (!isCorrectNetwork.value) {
    await switchNetwork()
    return
  }

  isMinting.value = true
  error.value = null

  try {
    // Implement actual minting logic here
    // Example placeholder for NFT contract interaction
    const contractAddress = '0x...' // Your zkSync NFT contract address
    const nftContract = await useNFTContract(contractAddress)
    
    const tx = await nftContract.mint(address.value, quantity.value)
    
    // Wait for transaction confirmation
    await tx.wait()

    // Success notification or redirect
    navigateTo('/success')
  } catch (err) {
    error.value = err instanceof Error 
      ? err.message 
      : 'Failed to mint NFT'
    console.error(err)
  } finally {
    isMinting.value = false
  }
}
</script>

<template>
  <div class="max-w-md mx-auto p-6 bg-gray-800/50 rounded-xl">
    <h1 class="text-2xl font-bold mb-6 text-primary-500">
      Mint NFT on zkSync
    </h1>

    <!-- Network Warning -->
    <div 
      v-if="status === 'connected' && !isCorrectNetwork" 
      class="mb-4 p-3 bg-yellow-500/10 text-yellow-400 rounded-lg"
    >
      Please switch to zkSync Sepolia Testnet
      <button 
        @click="switchNetwork" 
        class="ml-2 px-3 py-1 bg-yellow-500/20 rounded-md"
      >
        Switch Network
      </button>
    </div>

    <!-- Connection Section -->
    <div v-if="status === 'disconnected'" class="space-y-4">
      <p class="text-gray-400">Connect your wallet to mint NFTs</p>
      <div class="grid gap-3">
        <button
          v-for="connector in connectors"
          :key="connector.id"
          @click="connect({ connector, chainId })"
          class="w-full px-4 py-3 bg-primary-500 hover:bg-primary-600 text-white rounded-lg"
        >
          Connect with {{ connector.name }}
        </button>
      </div>
    </div>

    <!-- Minting Section -->
    <div v-else-if="isCorrectNetwork" class="space-y-4">
      <div>
        <label class="block text-gray-300 mb-2">
          Quantity to Mint
        </label>
        <input 
          v-model.number="quantity"
          type="number"
          min="1"
          max="5"
          class="w-full px-3 py-2 bg-gray-900/50 border border-gray-700 rounded-lg"
        />
      </div>

      <button
        @click="mintNFT"
        :disabled="isMinting"
        class="w-full px-4 py-3 bg-primary-500 hover:bg-primary-600 text-white rounded-lg disabled:opacity-50"
      >
        {{ isMinting ? 'Minting...' : 'Mint NFT' }}
      </button>

      <div v-if="error" class="mt-4 p-3 bg-red-500/10 text-red-400 rounded-lg">
        {{ error }}
      </div>
    </div>
  </div>
</template>

// File: composables/useNFTContract.ts
import { createPublicClient, createWalletClient, http } from 'viem'
import { zksyncSepoliaTestnet } from '@alpha/core/chains'

export async function useNFTContract(contractAddress: `0x${string}`) {
  const publicClient = createPublicClient({
    chain: zksyncSepoliaTestnet,
    transport: http('https://sepolia.era.zksync.dev')
  })

  const mintNFT = async (to: `0x${string}`, quantity: number) => {
    // Implement NFT minting logic 
    // This is a placeholder and should be replaced with actual contract interaction
    const txHash = await publicClient.writeContract({
      address: contractAddress,
      abi: NFT_CONTRACT_ABI, // Define your contract ABI
      functionName: 'mint',
      args: [to, quantity]
    })

    return { hash: txHash }
  }

  return {
    mint: mintNFT
  }
}

// NFT Contract ABI (placeholder)
const NFT_CONTRACT_ABI = [
  {
    inputs: [
      { name: 'to', type: 'address' },
      { name: 'quantity', type: 'uint256' }
    ],
    name: 'mint',
    outputs: [],
    stateMutability: 'payable',
    type: 'function'
  }
] as const
