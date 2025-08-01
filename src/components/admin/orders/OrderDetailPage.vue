<template>
    <div class="order-detail-page container mx-auto">
        <!-- Header Section -->
        <div class="header-section px-6 mt-3">
            <div class="flex justify-between items-start">
                <div class="flex items-center space-x-4">
                    <button @click="goBack" class="text-gray-600 hover:text-gray-800">
                        <i class="fas fa-arrow-left text-xl"></i>
                    </button>
                    <div>
                        <h1 class="text-2xl font-bold text-gray-900">Order #{{ orderId }}</h1>
                        <div class="flex items-center space-x-2 mt-1">
                            <span class="px-3 py-1 bg-green-100 text-green-800 rounded-full text-sm font-medium">
                                {{ orderStatusText }}
                            </span>
                            <span class="text-gray-500 text-sm">{{ formatDate(currentOrder?.created_at) }}</span>
                        </div>
                    </div>
                </div>
                <div class="flex items-center space-x-3">
                    <div class="relative">
                        <button
                            class="flex items-center px-4 py-2 bg-green-100 text-green-800 rounded-lg hover:bg-green-200">
                            <span>{{ orderStatusText }}</span>
                            <i class="fas fa-chevron-down ml-2"></i>
                        </button>
                    </div>
                    <button class="flex items-center px-4 py-2 bg-gray-100 text-gray-700 rounded-lg hover:bg-gray-200">
                        <i class="fas fa-print mr-2"></i>
                        Print
                    </button>
                    <button class="flex items-center px-4 py-2 bg-blue-100 text-blue-700 rounded-lg hover:bg-blue-200">
                        <i class="fas fa-edit mr-2"></i>
                        Edit
                    </button>
                </div>
            </div>
        </div>

        <!-- Loading State -->
        <div v-if="loading" class="flex justify-center items-center h-64">
            <div class="inline-block animate-spin rounded-full h-8 w-8 border-4 border-blue-500 border-t-transparent">
            </div>
        </div>

        <!-- Error State -->
        <div v-else-if="error" class="p-6 text-center text-red-500">
            {{ error }}
        </div>

        <!-- Main Content -->
        <div v-else-if="currentOrder" class="main-content p-6 min-h-screen">
            <div class="grid grid-cols-1 lg:grid-cols-2 gap-6">
                <!-- Left Column -->
                <div class="space-y-6">
                    <!-- Details Card -->
                    <div class="bg-white rounded-lg shadow p-6">
                        <div class="flex justify-between items-center mb-4">
                            <h3 class="text-lg font-semibold text-gray-900">Details</h3>
                            <button class="text-gray-500 hover:text-gray-700">
                                <i class="fas fa-edit"></i>
                            </button>
                        </div>

                        <!-- Order Items -->
                        <div class="space-y-4">
                            <div v-for="item in getOrderDetails()" :key="item.id" class="flex items-center space-x-4">
                                <img :src="item.variant?.product?.main_image?.image_path"
                                    class="w-12 h-12 object-cover rounded" :alt="item.variant?.product?.name" />
                                <div class="flex-1">
                                    <h4 class="font-medium text-gray-900">{{ item.variant?.product?.name }}</h4>
                                    <p class="text-sm text-gray-500">{{ item.variant?.sku }}</p>
                                </div>
                                <div class="text-right">
                                    <div class="text-sm text-gray-500">x{{ item.quantity }}</div>
                                    <div class="font-medium">{{ formatPrice(item.total_price) }}</div>
                                </div>
                            </div>
                        </div>

                        <!-- Price Summary -->
                        <div class="border-t border-gray-200 mt-4 pt-4 space-y-2">
                            <div class="flex justify-between">
                                <span class="text-gray-600">Subtotal</span>
                                <span>{{ formatPrice(currentOrder.total_price) }}</span>
                            </div>
                            <div class="flex justify-between text-red-600">
                                <span>Shipping</span>
                                <span>-{{ formatPrice(currentOrder.shipping_fee || 0) }}</span>
                            </div>
                            <div v-if="currentOrder.discount_price > 0" class="flex justify-between text-red-600">
                                <span>Discount</span>
                                <span>-{{ formatPrice(currentOrder.discount_price) }}</span>
                            </div>
                            <div class="flex justify-between">
                                <span class="text-gray-600">Taxes</span>
                                <span>{{ formatPrice(10) }}</span>
                            </div>
                            <div class="flex justify-between font-bold text-lg border-t border-gray-200 pt-2">
                                <span>Total</span>
                                <span>{{ formatPrice(currentOrder.final_price) }}</span>
                            </div>
                        </div>
                    </div>

                    <!-- History Card -->
                    <div class="bg-white rounded-lg shadow p-6">
                        <h3 class="text-lg font-semibold text-gray-900 mb-4">History</h3>
                        <div class="space-y-4">
                            <div class="flex items-start space-x-3">
                                <div class="w-3 h-3 bg-green-500 rounded-full mt-2"></div>
                                <div class="flex-1">
                                    <p class="font-medium text-gray-900">Delivery successful</p>
                                    <p class="text-sm text-gray-500">{{ formatDate(currentOrder.updated_at) }}</p>
                                </div>
                            </div>
                            <div class="flex items-start space-x-3">
                                <div class="w-3 h-3 bg-gray-300 rounded-full mt-2"></div>
                                <div class="flex-1">
                                    <p class="font-medium text-gray-900">Transporting to [2]</p>
                                    <p class="text-sm text-gray-500">{{ formatDate(currentOrder.created_at) }}</p>
                                </div>
                            </div>
                            <div class="flex items-start space-x-3">
                                <div class="w-3 h-3 bg-gray-300 rounded-full mt-2"></div>
                                <div class="flex-1">
                                    <p class="font-medium text-gray-900">Transporting to [1]</p>
                                    <p class="text-sm text-gray-500">{{ formatDate(currentOrder.created_at) }}</p>
                                </div>
                            </div>
                            <div class="flex items-start space-x-3">
                                <div class="w-3 h-3 bg-gray-300 rounded-full mt-2"></div>
                                <div class="flex-1">
                                    <p class="font-medium text-gray-900">The shipping unit has picked up the goods</p>
                                    <p class="text-sm text-gray-500">{{ formatDate(currentOrder.created_at) }}</p>
                                </div>
                            </div>
                            <div class="flex items-start space-x-3">
                                <div class="w-3 h-3 bg-gray-300 rounded-full mt-2"></div>
                                <div class="flex-1">
                                    <p class="font-medium text-gray-900">Order has been created</p>
                                    <p class="text-sm text-gray-500">{{ formatDate(currentOrder.created_at) }}</p>
                                </div>
                            </div>
                        </div>

                        <!-- Related Timestamps -->
                        <div class="mt-6 space-y-2">
                            <div class="flex justify-between text-sm">
                                <span class="text-gray-600">Order placed</span>
                                <span class="text-gray-900">{{ formatDate(currentOrder.created_at) }}</span>
                            </div>
                            <div class="flex justify-between text-sm">
                                <span class="text-gray-600">Payment time</span>
                                <span class="text-gray-900">{{ formatDate(currentOrder.created_at) }}</span>
                            </div>
                            <div class="flex justify-between text-sm">
                                <span class="text-gray-600">Delivery time for the carrier</span>
                                <span class="text-gray-900">{{ formatDate(currentOrder.updated_at) }}</span>
                            </div>
                            <div class="flex justify-between text-sm">
                                <span class="text-gray-600">Completion time</span>
                                <span class="text-gray-900">{{ formatDate(currentOrder.updated_at) }}</span>
                            </div>
                        </div>

                        <button class="text-blue-600 text-sm mt-4 hover:text-blue-800">
                            Show more >
                        </button>
                    </div>
                </div>

                <!-- Right Column -->
                <div class="space-y-6">
                    <!-- Customer Info Card -->
                    <div class="bg-white rounded-lg shadow p-6">
                        <div class="flex justify-between items-center mb-4">
                            <h3 class="text-lg font-semibold text-gray-900">Customer info</h3>
                            <button class="text-gray-500 hover:text-gray-700">
                                <i class="fas fa-edit"></i>
                            </button>
                        </div>
                        <div class="flex items-center space-x-4">
                            <div class="w-12 h-12 bg-gray-300 rounded-full flex items-center justify-center">
                                <i class="fas fa-user text-gray-600"></i>
                            </div>
                            <div class="flex-1">
                                <h4 class="font-medium text-gray-900">{{ currentOrder.user?.username }}</h4>
                                <p class="text-sm text-gray-500">{{ currentOrder.user?.email }}</p>
                                <p class="text-sm text-gray-500">IP: 192.158.1.38</p>
                            </div>
                        </div>
                        <button class="text-red-600 text-sm mt-3 hover:text-red-800">
                            + Add to blacklist
                        </button>
                    </div>

                    <!-- Delivery Card -->
                    <div class="bg-white rounded-lg shadow p-6">
                        <div class="flex justify-between items-center mb-4">
                            <h3 class="text-lg font-semibold text-gray-900">Delivery</h3>
                            <button class="text-gray-500 hover:text-gray-700">
                                <i class="fas fa-edit"></i>
                            </button>
                        </div>
                        <div class="space-y-2">
                            <div class="flex justify-between">
                                <span class="text-gray-600">Ship by:</span>
                                <span class="font-medium">DHL</span>
                            </div>
                            <div class="flex justify-between">
                                <span class="text-gray-600">Speedy:</span>
                                <span class="font-medium">Standard</span>
                            </div>
                            <div class="flex justify-between">
                                <span class="text-gray-600">Tracking No.:</span>
                                <a href="#" class="text-blue-600 underline font-medium">{{ currentOrder.tracking_code ||
                                    'SPX037739199373' }}</a>
                            </div>
                        </div>
                    </div>

                    <!-- Shipping Card -->
                    <div class="bg-white rounded-lg shadow p-6">
                        <div class="flex justify-between items-center mb-4">
                            <h3 class="text-lg font-semibold text-gray-900">Shipping</h3>
                            <button class="text-gray-500 hover:text-gray-700">
                                <i class="fas fa-edit"></i>
                            </button>
                        </div>
                        <div class="space-y-2">
                            <p class="text-gray-900">{{ getFullAddress(currentOrder.address) }}</p>
                            <p class="text-gray-600">{{ currentOrder.address?.phone }}</p>
                        </div>
                    </div>

                    <!-- Payment Card -->
                    <div class="bg-white rounded-lg shadow p-6">
                        <div class="flex justify-between items-center mb-4">
                            <h3 class="text-lg font-semibold text-gray-900">Payment</h3>
                            <button class="text-gray-500 hover:text-gray-700">
                                <i class="fas fa-edit"></i>
                            </button>
                        </div>
                        <div class="flex items-center space-x-3">
                            <div class="flex-1">
                                <p class="font-medium text-gray-900">**** **** **** 5678</p>
                                <p class="text-sm text-gray-500">{{ getPaymentMethod(currentOrder.payment_method) }}</p>
                            </div>
                            <div class="w-8 h-8 bg-blue-100 rounded flex items-center justify-center">
                                <i class="fab fa-cc-mastercard text-blue-600"></i>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<script setup>
import { ref, onMounted, computed } from 'vue'
import { useRouter } from 'vue-router'
import { useOrder } from '../../../composable/useOrder'

const props = defineProps({
    orderId: {
        type: [String, Number],
        required: true
    }
})

const router = useRouter()

const {
    currentOrder,
    loading,
    error,
    getOrder,
    getOrderStatus,
    getPaymentMethod,
    formatPrice
} = useOrder()

const orderStatusText = computed(() => {
    if (!currentOrder.value?.status) return ''
    return getOrderStatus(currentOrder.value.status)
})

const goBack = () => {
    router.push('/admin/orders')
}

const formatDate = (date) => {
    if (!date) return ''
    return new Date(date).toLocaleDateString('en-US', {
        year: 'numeric',
        month: 'short',
        day: 'numeric',
        hour: '2-digit',
        minute: '2-digit'
    })
}

const getFullAddress = (address) => {
    if (!address) return '19034 Verna Unions Apt. 164 -Honolulu, RI / 87535'
    const parts = [
        address.street,
        address.hamlet,
        address.ward,
        address.district,
        address.province
    ].filter(Boolean)
    return parts.join(', ')
}

const getOrderDetails = () => {
    return currentOrder.value?.orderDetails || currentOrder.value?.order_details || []
}

onMounted(async () => {
    await getOrder(props.orderId)
})
</script>

<style scoped>
.order-detail-page {
    min-height: 100vh;
}

.header-section {
    position: sticky;
    top: 0;
}

.main-content {
    padding-top: 1.5rem;
}
</style>