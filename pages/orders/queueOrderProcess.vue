<template>
  <div class="container mx-auto overflow-hidden min-w-full">
    <!-- Header -->
    <div class="flex items-center justify-between px-4">
      <HeadersContent
        title="Queue Order Management"
        description="Manage order priorities using Greedy Algorithm"
      />
      <div class="flex gap-2">
        <Button @click="recalculatePriorities" variant="outline">
          <RefreshCw class="mr-2 h-4 w-4" />
          Recalculate Priorities
        </Button>
        <Button @click="showCapacityModal = true" class="bg-primary">
          <Settings class="mr-2 h-4 w-4" />
          Capacity Settings
        </Button>
      </div>
    </div>

    <!-- Statistics Cards -->
    <div class="grid grid-cols-1 md:grid-cols-4 gap-4 mb-6 px-4">
      <!-- Total Antrian -->
      <Card>
        <CardHeader
          class="flex flex-row items-center justify-between space-y-0"
        >
          <CardTitle class="text-sm font-medium"> Total Antrian </CardTitle>
          <Clock class="text-blue-500" />
        </CardHeader>
        <CardContent>
          <div class="text-xl font-bold">{{ queueOrders.length }}</div>
          <p class="text-xs text-muted-foreground">Queue Orders</p>
        </CardContent>
      </Card>

      <!-- High Priority Count atau prioritas Utama -->
      <Card>
        <CardHeader
          class="flex flex-row items-center justify-between space-y-0"
        >
          <CardTitle class="text-sm font-medium"> Prioritas Utama </CardTitle>
          <AlertTriangle class="text-red-500" />
        </CardHeader>
        <CardContent>
          <div class="text-xl font-bold">{{ highPriorityCount }}</div>
          <p class="text-xs text-muted-foreground">High Priority</p>
        </CardContent>
      </Card>

      <!-- Today's Capacity atau total kapasistas -->
      <Card>
        <CardHeader
          class="flex flex-row items-center justify-between space-y-0"
        >
          <CardTitle class="text-sm font-medium">Kapasitas Hari Ini</CardTitle>
          <TrendingUp class="h-8 w-8 text-green-500" />
        </CardHeader>
        <CardContent>
          <div class="text-xl font-bold">
            {{ todayCapacity.current }}/{{ todayCapacity.max }}
          </div>
          <p class="text-xs text-muted-foreground">Today's Capacity</p>
        </CardContent>
      </Card>

      <!-- Average Processing Time atau rata rata waktu proses -->
      <Card>
        <CardHeader
          class="flex flex-row items-center justify-between space-y-0"
        >
          <CardTitle class="text-sm font-medium">
            Rata-rata Waktu Proses
          </CardTitle>
          <Timer class="text-purple-500" />
        </CardHeader>
        <CardContent>
          <div class="text-xl font-bold">{{ averageProcessingTime }} jam</div>
          <p class="text-xs text-muted-foreground">Average Processing Time</p>
        </CardContent>
      </Card>

      <!-- <Card>
        <CardContent class="p-4">
          <div class="flex items-center justify-between">
            <div>
              <p class="text-sm font-medium text-muted-foreground">
                Avg Processing Time
              </p>
              <p class="text-2xl font-bold">{{ averageProcessingTime }}h</p>
            </div>
            <Timer class="h-8 w-8 text-purple-500" />
          </div>
        </CardContent>
      </Card> -->
    </div>

    <!-- Priority Algorithm Info -->
    <Card class="mb-6 mx-4">
      <CardHeader>
        <CardTitle class="flex items-center">
          <Brain class="mr-2 h-5 w-5" />
          Greedy Algorithm Priority Calculation
        </CardTitle>
      </CardHeader>
      <CardContent>
        <div class="grid grid-cols-1 md:grid-cols-5 gap-4 text-sm">
          <div class="text-center">
            <div class="font-semibold text-red-600">Deadline Score</div>
            <div class="text-muted-foreground">40% weight</div>
            <div class="text-xs mt-1">Urgent orders first</div>
          </div>
          <div class="text-center">
            <div class="font-semibold text-green-600">Value Score</div>
            <div class="text-muted-foreground">25% weight</div>
            <div class="text-xs mt-1">High-value priority</div>
          </div>
          <div class="text-center">
            <div class="font-semibold text-blue-600">Complexity Score</div>
            <div class="text-muted-foreground">20% weight</div>
            <div class="text-xs mt-1">Simple products first</div>
          </div>
          <div class="text-center">
            <div class="font-semibold text-purple-600">Customer Score</div>
            <div class="text-muted-foreground">10% weight</div>
            <div class="text-xs mt-1">VIP customers</div>
          </div>
          <div class="text-center">
            <div class="font-semibold text-orange-600">Time Score</div>
            <div class="text-muted-foreground">5% weight</div>
            <div class="text-xs mt-1">FIFO tie-breaker</div>
          </div>
        </div>
      </CardContent>
    </Card>

    <!-- Queue Table -->
    <div class="border max-w-screen overflow-hidden">
      <div class="">
        <Table>
          <TableHeader>
            <TableRow>
              <TableHead class="w-16 min-w-16">Queue #</TableHead>
              <TableHead class="w-24 min-w-24">Order ID</TableHead>
              <TableHead class="min-w-48">Customer</TableHead>
              <TableHead class="min-w-48">Products</TableHead>
              <TableHead class="min-w-40">Deadline</TableHead>
              <TableHead class="min-w-32">Value</TableHead>
              <TableHead class="min-w-48">Priority Score</TableHead>
              <TableHead class="min-w-24">Status</TableHead>
              <TableHead class="min-w-32">Actions</TableHead>
            </TableRow>
          </TableHeader>
          <TableBody>
            <TableRow
              v-for="(order, index) in sortedQueueOrders"
              :key="order.id"
              :class="getPriorityRowClass(order.priorityScore)"
            >
              <!-- Queue Position -->
              <TableCell class="font-bold text-center">
                <Badge :variant="getQueuePositionVariant(index + 1)">
                  {{ index + 1 }}
                </Badge>
              </TableCell>

              <!-- Order ID -->
              <TableCell class="font-mono text-sm">
                {{ order.id }}
              </TableCell>

              <!-- Customer -->
              <TableCell>
                <div class="flex items-center">
                  <div
                    class="h-8 w-8 rounded-full bg-gray-200 flex items-center justify-center mr-3 flex-shrink-0"
                  >
                    <span class="text-sm font-medium text-gray-600">
                      {{ order.customer.name.charAt(0).toUpperCase() }}
                    </span>
                  </div>
                  <div class="min-w-0">
                    <div class="font-medium whitespace-nowrap">
                      {{ order.customer.name }}
                    </div>
                    <Badge
                      v-if="order.customer.type === 'vip'"
                      variant="secondary"
                      class="text-xs"
                    >
                      VIP
                    </Badge>
                  </div>
                </div>
              </TableCell>

              <!-- Products -->
              <TableCell>
                <div class="space-y-1">
                  <div
                    v-for="product in order.products.slice(0, 2)"
                    :key="product.id"
                    class="text-sm whitespace-nowrap"
                  >
                    {{ product.name }} x{{ product.quantity }}
                  </div>
                  <div
                    v-if="order.products.length > 2"
                    class="text-xs text-muted-foreground"
                  >
                    +{{ order.products.length - 2 }} more items
                  </div>
                </div>
              </TableCell>

              <!-- Deadline -->
              <TableCell>
                <div class="text-sm">
                  <div
                    :class="getDeadlineClass(order.deadline)"
                    class="whitespace-nowrap"
                  >
                    {{ formatDate(order.deadline) }}
                  </div>
                  <div class="text-xs text-muted-foreground whitespace-nowrap">
                    {{ getTimeUntilDeadline(order.deadline) }}
                  </div>
                </div>
              </TableCell>

              <!-- Value -->
              <TableCell class="font-medium">
                <div class="whitespace-nowrap">
                  Rp {{ formatPrice(order.total) }}
                </div>
              </TableCell>

              <!-- Priority Score -->
              <TableCell>
                <div class="flex items-center space-x-2 min-w-0">
                  <div class="flex-1 min-w-20">
                    <div class="w-full bg-gray-200 rounded-full h-2">
                      <div
                        class="h-2 rounded-full transition-all duration-300"
                        :class="getPriorityBarClass(order.priorityScore)"
                        :style="{
                          width: `${(order.priorityScore / 100) * 100}%`,
                        }"
                      ></div>
                    </div>
                  </div>
                  <span class="text-sm font-bold whitespace-nowrap">{{
                    order.priorityScore.toFixed(1)
                  }}</span>
                </div>
                <div
                  class="text-xs text-muted-foreground mt-1 whitespace-nowrap"
                >
                  D:{{ order.scores.deadline }} V:{{ order.scores.value }} C:{{
                    order.scores.complexity
                  }}
                  Cu:{{ order.scores.customer }}
                </div>
              </TableCell>

              <!-- Status -->
              <TableCell>
                <Badge :variant="getStatusVariant(order.status)">
                  {{ order.status }}
                </Badge>
              </TableCell>

              <!-- Actions -->
              <TableCell>
                <div class="flex items-center space-x-2">
                  <Button
                    variant="ghost"
                    size="sm"
                    @click="viewOrderDetails(order)"
                    title="View Details"
                  >
                    <Eye class="h-4 w-4" />
                  </Button>
                  <Button
                    variant="ghost"
                    size="sm"
                    @click="startProcessing(order.id)"
                    title="Start Processing"
                    :disabled="order.status === 'processing'"
                  >
                    <Play class="h-4 w-4" />
                  </Button>
                  <Button
                    variant="ghost"
                    size="sm"
                    @click="moveToTop(order.id)"
                    title="Move to Top"
                    class="text-orange-600"
                  >
                    <ArrowUp class="h-4 w-4" />
                  </Button>
                </div>
              </TableCell>
            </TableRow>
          </TableBody>
        </Table>
      </div>
    </div>

    <!-- Order Details Modal -->
    <Dialog v-model:open="showDetailsModal">
      <DialogContent class="max-w-3xl overflow-auto max-h-[90vh]">
        <DialogHeader>
          <DialogTitle>Order Details & Priority Breakdown</DialogTitle>
        </DialogHeader>
        <div v-if="selectedOrder" class="space-y-6">
          <!-- Basic Info -->
          <div class="grid grid-cols-2 gap-4">
            <div>
              <Label class="font-medium">Order ID</Label>
              <p>{{ selectedOrder.id }}</p>
            </div>
            <div>
              <Label class="font-medium">Customer</Label>
              <p>
                {{ selectedOrder.customer.name }}
                <Badge
                  v-if="selectedOrder.customer.type === 'vip'"
                  variant="secondary"
                  class="ml-2"
                  >VIP</Badge
                >
              </p>
            </div>
            <div>
              <Label class="font-medium">Total Value</Label>
              <p class="font-bold">Rp {{ formatPrice(selectedOrder.total) }}</p>
            </div>
            <div>
              <Label class="font-medium">Deadline</Label>
              <p :class="getDeadlineClass(selectedOrder.deadline)">
                {{ formatDate(selectedOrder.deadline) }}
              </p>
            </div>
          </div>

          <!-- Priority Score Breakdown -->
          <div>
            <Label class="font-medium mb-3 block"
              >Priority Score Breakdown</Label
            >
            <div class="space-y-3">
              <div
                class="flex items-center justify-between p-3 bg-red-50 rounded"
              >
                <span>Deadline Score (40%)</span>
                <div class="flex items-center space-x-2">
                  <div class="w-32 bg-gray-200 rounded-full h-2">
                    <div
                      class="bg-red-500 h-2 rounded-full"
                      :style="{ width: `${selectedOrder.scores.deadline}%` }"
                    ></div>
                  </div>
                  <span class="font-bold"
                    >{{ selectedOrder.scores.deadline }}/40</span
                  >
                </div>
              </div>
              <div
                class="flex items-center justify-between p-3 bg-green-50 rounded"
              >
                <span>Value Score (25%)</span>
                <div class="flex items-center space-x-2">
                  <div class="w-32 bg-gray-200 rounded-full h-2">
                    <div
                      class="bg-green-500 h-2 rounded-full"
                      :style="{
                        width: `${(selectedOrder.scores.value / 25) * 100}%`,
                      }"
                    ></div>
                  </div>
                  <span class="font-bold"
                    >{{ selectedOrder.scores.value }}/25</span
                  >
                </div>
              </div>
              <div
                class="flex items-center justify-between p-3 bg-blue-50 rounded"
              >
                <span>Complexity Score (20%)</span>
                <div class="flex items-center space-x-2">
                  <div class="w-32 bg-gray-200 rounded-full h-2">
                    <div
                      class="bg-blue-500 h-2 rounded-full"
                      :style="{
                        width: `${
                          (selectedOrder.scores.complexity / 20) * 100
                        }%`,
                      }"
                    ></div>
                  </div>
                  <span class="font-bold"
                    >{{ selectedOrder.scores.complexity }}/20</span
                  >
                </div>
              </div>
              <div
                class="flex items-center justify-between p-3 bg-purple-50 rounded"
              >
                <span>Customer Score (10%)</span>
                <div class="flex items-center space-x-2">
                  <div class="w-32 bg-gray-200 rounded-full h-2">
                    <div
                      class="bg-purple-500 h-2 rounded-full"
                      :style="{
                        width: `${(selectedOrder.scores.customer / 10) * 100}%`,
                      }"
                    ></div>
                  </div>
                  <span class="font-bold"
                    >{{ selectedOrder.scores.customer }}/10</span
                  >
                </div>
              </div>
              <div
                class="flex items-center justify-between p-3 bg-orange-50 rounded"
              >
                <span>Time Score (5%)</span>
                <div class="flex items-center space-x-2">
                  <div class="w-32 bg-gray-200 rounded-full h-2">
                    <div
                      class="bg-orange-500 h-2 rounded-full"
                      :style="{
                        width: `${(selectedOrder.scores.time / 5) * 100}%`,
                      }"
                    ></div>
                  </div>
                  <span class="font-bold"
                    >{{ selectedOrder.scores.time }}/5</span
                  >
                </div>
              </div>
              <div
                class="flex items-center justify-between p-3 bg-gray-100 rounded border-2 border-gray-300"
              >
                <span class="font-bold">Total Priority Score</span>
                <span class="text-xl font-bold"
                  >{{ selectedOrder.priorityScore.toFixed(1) }}/100</span
                >
              </div>
            </div>
          </div>

          <!-- Products List -->
          <div>
            <Label class="font-medium mb-3 block">Products</Label>
            <div class="space-y-2">
              <div
                v-for="product in selectedOrder.products"
                :key="product.id"
                class="flex justify-between items-center p-2 bg-gray-50 rounded"
              >
                <div>
                  <span class="font-medium">{{ product.name }}</span>
                  <span class="text-sm text-muted-foreground ml-2">
                    (Complexity: {{ product.complexity }}/10)
                  </span>
                </div>
                <div class="text-right">
                  <div>Qty: {{ product.quantity }}</div>
                  <div class="text-sm text-muted-foreground">
                    Rp {{ formatPrice(product.subtotal) }}
                  </div>
                </div>
              </div>
            </div>
          </div>
        </div>
      </DialogContent>
    </Dialog>

    <!-- Capacity Settings Modal -->
    <Dialog v-model:open="showCapacityModal">
      <DialogContent>
        <DialogHeader>
          <DialogTitle>Production Capacity Settings</DialogTitle>
        </DialogHeader>
        <div class="space-y-4">
          <div>
            <Label for="maxDaily">Maximum Orders Per Day</Label>
            <Input
              id="maxDaily"
              v-model="capacitySettings.maxOrdersPerDay"
              type="number"
              min="1"
            />
          </div>
          <div>
            <Label for="avgTime">Average Processing Time (hours)</Label>
            <Input
              id="avgTime"
              v-model="capacitySettings.avgProcessingTime"
              type="number"
              min="0.5"
              step="0.5"
            />
          </div>
          <div class="flex justify-end space-x-2">
            <Button variant="outline" @click="showCapacityModal = false"
              >Cancel</Button
            >
            <Button @click="saveCapacitySettings">Save Settings</Button>
          </div>
        </div>
      </DialogContent>
    </Dialog>
  </div>
</template>

<script setup>
import HeadersContent from "~/components/ui/HeadersContent.vue";
import { ref, computed, onMounted } from "vue";
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";
import { Label } from "@/components/ui/label";
import { Card, CardContent, CardHeader, CardTitle } from "@/components/ui/card";
import {
  Table,
  TableBody,
  TableCell,
  TableHead,
  TableHeader,
  TableRow,
} from "@/components/ui/table";
import { Badge } from "@/components/ui/badge";
import {
  Dialog,
  DialogContent,
  DialogHeader,
  DialogTitle,
} from "@/components/ui/dialog";
import {
  RefreshCw,
  Settings,
  Clock,
  AlertTriangle,
  TrendingUp,
  Timer,
  Brain,
  Eye,
  Play,
  ArrowUp,
} from "lucide-vue-next";

// Sample queue orders data
const queueOrders = ref([
  {
    id: "ORD-001",
    customer: { name: "Siti Aminah", type: "vip" },
    products: [
      {
        id: "P1",
        name: "Wedding Cake 3 Tier",
        quantity: 1,
        complexity: 9,
        subtotal: 2500000,
      },
      {
        id: "P2",
        name: "Cupcakes",
        quantity: 50,
        complexity: 3,
        subtotal: 500000,
      },
    ],
    total: 3000000,
    deadline: "2024-01-20T10:00:00Z",
    orderTime: "2024-01-15T08:00:00Z",
    status: "queue",
    priorityScore: 0,
    scores: { deadline: 0, value: 0, complexity: 0, customer: 0, time: 0 },
  },
  {
    id: "ORD-002",
    customer: { name: "Budi Santoso", type: "regular" },
    products: [
      {
        id: "P3",
        name: "Birthday Cake",
        quantity: 1,
        complexity: 5,
        subtotal: 350000,
      },
    ],
    total: 350000,
    deadline: "2024-01-18T15:00:00Z",
    orderTime: "2024-01-16T10:00:00Z",
    status: "queue",
    priorityScore: 0,
    scores: { deadline: 0, value: 0, complexity: 0, customer: 0, time: 0 },
  },
  {
    id: "ORD-003",
    customer: { name: "Maya Sari", type: "vip" },
    products: [
      {
        id: "P4",
        name: "Donat",
        quantity: 24,
        complexity: 2,
        subtotal: 120000,
      },
    ],
    total: 120000,
    deadline: "2024-01-17T12:00:00Z",
    orderTime: "2024-01-16T14:00:00Z",
    status: "queue",
    priorityScore: 0,
    scores: { deadline: 0, value: 0, complexity: 0, customer: 0, time: 0 },
  },
  {
    id: "ORD-004",
    customer: { name: "Ahmad Rizki", type: "regular" },
    products: [
      {
        id: "P5",
        name: "Kue Ulang Tahun Custom",
        quantity: 1,
        complexity: 8,
        subtotal: 800000,
      },
    ],
    total: 800000,
    deadline: "2024-01-19T16:00:00Z",
    orderTime: "2024-01-15T16:00:00Z",
    status: "queue",
    priorityScore: 0,
    scores: { deadline: 0, value: 0, complexity: 0, customer: 0, time: 0 },
  },
  {
    id: "ORD-005",
    customer: { name: "Dewi Lestari", type: "regular" },
    products: [
      {
        id: "P6",
        name: "Roti Tawar",
        quantity: 5,
        complexity: 1,
        subtotal: 75000,
      },
    ],
    total: 75000,
    deadline: "2024-01-21T09:00:00Z",
    orderTime: "2024-01-16T11:00:00Z",
    status: "queue",
    priorityScore: 0,
    scores: { deadline: 0, value: 0, complexity: 0, customer: 0, time: 0 },
  },
]);

// State
const showDetailsModal = ref(false);
const showCapacityModal = ref(false);
const selectedOrder = ref(null);
const capacitySettings = ref({
  maxOrdersPerDay: 20,
  avgProcessingTime: 4,
});

const todayCapacity = ref({
  current: 8,
  max: 20,
});

// Computed
const sortedQueueOrders = computed(() => {
  return [...queueOrders.value].sort(
    (a, b) => b.priorityScore - a.priorityScore
  );
});

const highPriorityCount = computed(() => {
  return queueOrders.value.filter((order) => order.priorityScore >= 70).length;
});

const averageProcessingTime = computed(() => {
  return capacitySettings.value.avgProcessingTime;
});

// Greedy Algorithm Implementation
const calculatePriorityScore = (order) => {
  const now = new Date();
  const deadline = new Date(order.deadline);
  const orderTime = new Date(order.orderTime);

  // 1. Deadline Score (40% weight) - Greedy: Most urgent first
  const hoursUntilDeadline = (deadline - now) / (1000 * 60 * 60);
  let deadlineScore = 0;
  if (hoursUntilDeadline <= 24) deadlineScore = 40;
  else if (hoursUntilDeadline <= 48) deadlineScore = 30;
  else if (hoursUntilDeadline <= 72) deadlineScore = 20;
  else deadlineScore = 10;

  // 2. Value Score (25% weight) - Greedy: Highest value first
  let valueScore = 0;
  if (order.total >= 2000000) valueScore = 25;
  else if (order.total >= 1000000) valueScore = 20;
  else if (order.total >= 500000) valueScore = 15;
  else if (order.total >= 200000) valueScore = 10;
  else valueScore = 5;

  // 3. Complexity Score (20% weight) - Greedy: Simplest first for throughput
  const avgComplexity =
    order.products.reduce((sum, p) => sum + p.complexity, 0) /
    order.products.length;
  let complexityScore = 0;
  if (avgComplexity <= 3) complexityScore = 20;
  else if (avgComplexity <= 5) complexityScore = 15;
  else if (avgComplexity <= 7) complexityScore = 10;
  else complexityScore = 5;

  // 4. Customer Score (10% weight) - Greedy: VIP first
  const customerScore = order.customer.type === "vip" ? 10 : 5;

  // 5. Time Score (5% weight) - FIFO tie-breaker
  const hoursFromOrder = (now - orderTime) / (1000 * 60 * 60);
  const timeScore = Math.min(5, (hoursFromOrder / 24) * 5);

  const totalScore =
    deadlineScore + valueScore + complexityScore + customerScore + timeScore;

  return {
    total: totalScore,
    deadline: deadlineScore,
    value: valueScore,
    complexity: complexityScore,
    customer: customerScore,
    time: Math.round(timeScore * 10) / 10,
  };
};

// Methods
const recalculatePriorities = () => {
  queueOrders.value.forEach((order) => {
    const scores = calculatePriorityScore(order);
    order.priorityScore = scores.total;
    order.scores = {
      deadline: scores.deadline,
      value: scores.value,
      complexity: scores.complexity,
      customer: scores.customer,
      time: scores.time,
    };
  });
};

const formatPrice = (price) => {
  return new Intl.NumberFormat("id-ID").format(price);
};

const formatDate = (dateString) => {
  return new Date(dateString).toLocaleDateString("id-ID", {
    year: "numeric",
    month: "short",
    day: "numeric",
    hour: "2-digit",
    minute: "2-digit",
  });
};

const getTimeUntilDeadline = (deadline) => {
  const now = new Date();
  const deadlineDate = new Date(deadline);
  const diffHours = Math.ceil((deadlineDate - now) / (1000 * 60 * 60));

  if (diffHours < 0) return "Overdue";
  if (diffHours < 24) return `${diffHours}h left`;
  const days = Math.ceil(diffHours / 24);
  return `${days}d left`;
};

const getDeadlineClass = (deadline) => {
  const now = new Date();
  const deadlineDate = new Date(deadline);
  const diffHours = (deadlineDate - now) / (1000 * 60 * 60);

  if (diffHours < 0) return "text-red-600 font-bold";
  if (diffHours <= 24) return "text-red-500 font-medium";
  if (diffHours <= 48) return "text-orange-500";
  return "text-gray-700";
};

const getPriorityRowClass = (score) => {
  if (score >= 80) return "bg-red-50 border-l-4 border-red-500";
  if (score >= 60) return "bg-orange-50 border-l-4 border-orange-500";
  if (score >= 40) return "bg-yellow-50 border-l-4 border-yellow-500";
  return "hover:bg-muted/50";
};

const getPriorityBarClass = (score) => {
  if (score >= 80) return "bg-red-500";
  if (score >= 60) return "bg-orange-500";
  if (score >= 40) return "bg-yellow-500";
  return "bg-green-500";
};

const getQueuePositionVariant = (position) => {
  if (position <= 3) return "destructive";
  if (position <= 6) return "secondary";
  return "outline";
};

const getStatusVariant = (status) => {
  switch (status) {
    case "queue":
      return "secondary";
    case "processing":
      return "default";
    case "completed":
      return "outline";
    default:
      return "secondary";
  }
};

const viewOrderDetails = (order) => {
  selectedOrder.value = order;
  showDetailsModal.value = true;
};

const startProcessing = (orderId) => {
  const order = queueOrders.value.find((o) => o.id === orderId);
  if (order) {
    order.status = "processing";
  }
};

const moveToTop = (orderId) => {
  const order = queueOrders.value.find((o) => o.id === orderId);
  if (order) {
    order.priorityScore = 100; // Force to top
  }
};

const saveCapacitySettings = () => {
  todayCapacity.value.max = capacitySettings.value.maxOrdersPerDay;
  showCapacityModal.value = false;
};

// Initialize
onMounted(() => {
  recalculatePriorities();
});
</script>

<style scoped>
/* Custom animations for priority changes */
.priority-animation {
  transition: all 0.3s ease-in-out;
}
</style>
