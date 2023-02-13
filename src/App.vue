<template>
  <div class="min-h-full">
    <Disclosure as="nav" class="bg-indigo-600" v-slot="{ open }">
      <div class="mx-auto max-w-7xl px-4 sm:px-6 lg:px-8">
        <div class="flex h-16 items-center justify-between">
          <div class="flex items-center">
            <div class="hidden md:block">
              <div class="flex items-baseline space-x-4">
                <a v-for="item in navigation" :key="item.name" :href="item.href" :class="[item.current ? 'bg-indigo-700 text-white' : 'text-white hover:bg-indigo-500 hover:bg-opacity-75', 'px-3 py-2 rounded-md text-sm font-medium']" :aria-current="item.current ? 'page' : undefined">{{ item.name }}</a>
              </div>
            </div>
          </div>

          <div v-if="mode == 'list'">
            <button type="button" class="py-1 px-3 border bg-gray-100">Agregar cliente</button>
          </div>
        </div>
      </div>

      <DisclosurePanel class="md:hidden">
        <div class="space-y-1 px-2 pt-2 pb-3 sm:px-3">
          <DisclosureButton v-for="item in navigation" :key="item.name" as="a" :href="item.href" :class="[item.current ? 'bg-indigo-700 text-white' : 'text-white hover:bg-indigo-500 hover:bg-opacity-75', 'block px-3 py-2 rounded-md text-base font-medium']" :aria-current="item.current ? 'page' : undefined">{{ item.name }}</DisclosureButton>
        </div>
      </DisclosurePanel>
    </Disclosure>

    <header class="bg-white shadow">
      <div class="mx-auto max-w-7xl py-6 px-4 sm:px-6 lg:px-8">
        <h1 v-if="mode == 'list'" class="text-3xl font-bold leading-tight tracking-tight text-gray-900">Clientes</h1>
        <div v-if="mode == 'profile'" class="flex items-center space-x-3">
          <button type="button" class="py-1 px-3 border bg-gray-100" @click="unselectCustomer">&lt Volver</button>
          <h1 class="text-3xl font-bold leading-tight tracking-tight text-gray-900">{{ customerSelected.firstname }} {{ customerSelected.lastname }}</h1>
        </div>
      </div>
    </header>

    <main v-if="mode == 'list'">
      <div class="mx-auto max-w-7xl py-3 sm:px-6 lg:px-8">
        <div class="flex flex-col space-y-3">
          <div class="px-4 sm:px-0">
            <div>
              <dl class="grid grid-cols-1 gap-5 sm:grid-cols-3">
                <div v-for="item in stats" :key="item.name" class="overflow-hidden rounded-lg bg-white px-4 py-5 shadow sm:p-6">
                  <dt class="truncate text-sm font-medium text-black">{{ item.name }}</dt>
                  <dd class="mt-1 text-3xl font-semibold tracking-tight text-gray-900">{{ item.stat }}</dd>
                </div>
              </dl>
            </div>
          </div>

          <div class="my-1">
            <input v-model="formKeyword" @keyup.enter="searchCustomer" type="text" name="search" class="w-32 border p-1" placeholder="Buscar cliente por nombre y apellido" />
            <button type="button" class="py-1 px-3 border bg-gray-100" @click="searchCustomer">Buscar</button>
          </div>

          <div class="overflow-hidden bg-white shadow sm:rounded-md">
            <ul role="list" class="divide-y divide-gray-200">
              <li v-for="customer in customers" :key="customer.email">
                <div class="block">
                  <div class="flex items-center px-4 py-4 sm:px-6">
                    <div class="flex min-w-0 flex-1 items-center">
                      <div class="flex-shrink-0">
                        <img class="h-12 w-12 rounded-full" :src="customer.photo" alt="" />
                      </div>
                      <div class="min-w-0 flex-1 px-4 md:grid md:grid-cols-3 md:gap-4">
                        <div>
                          <p class="truncate text-sm text-black">{{ customer.firstname }} {{ customer.lastname }}</p>
                          <p class="mt-2 flex items-center text-sm text-black">
                            <EnvelopeIcon class="mr-1.5 h-5 w-5 flex-shrink-0 text-gray-400" aria-hidden="true" />
                            <span class="truncate">{{ customer.email }}</span>
                          </p>
                          <p class="mt-2 flex items-center text-sm text-black">
                            <PhoneIcon class="mr-1.5 h-5 w-5 flex-shrink-0 text-gray-400" aria-hidden="true" />
                            <span class="truncate">{{ customer.cellphone }}</span>
                          </p>
                        </div>
                        <div class="hidden md:block">
                          <div>
                            <p class="text-sm text-gray-900">
                              Birthdate
                              {{ ' ' }}
                              <time :datetime="customer.date">{{ customer.birthdate }}</time>
                            </p>
                            <template v-if="customer.status == 'active'">
                              <p class="mt-2 flex items-center text-sm text-black">
                                <CheckCircleIcon class="mr-1.5 h-5 w-5 flex-shrink-0 text-green-400" aria-hidden="true" />
                                {{ customer.status }}
                              </p>
                            </template>
                            <template v-if="customer.status == 'inactive'">
                              <p class="mt-2 flex items-center text-sm text-black">
                                <MinusCircleIcon class="mr-1.5 h-5 w-5 flex-shrink-0 text-yellow-400" aria-hidden="true" />
                                {{ customer.status }}
                              </p>
                            </template>
                            <template v-if="customer.status == 'blocked'">
                              <p class="mt-2 flex items-center text-sm text-black">
                                <XCircleIcon class="mr-1.5 h-5 w-5 flex-shrink-0 text-red-400" aria-hidden="true" />
                                {{ customer.status }}
                              </p>
                            </template>
                          </div>
                        </div>
                        <div class="hidden md:block">
                          <div>
                            <p class="truncate text-sm text-black">Personas conocidas</p>
                            <div class="flex -space-x-1 overflow-hidden">
                              <img v-for="person of customer.people_known" class="inline-block h-6 w-6 rounded-full ring-2 ring-white" :src="person.photo" :title="`${person.firstname} ${person.lastname}`" alt="" />
                            </div>
                          </div>
                          <div>
                            <p class="truncate text-sm text-black">Personas desconocidas</p>
                            <div class="flex -space-x-1 overflow-hidden">
                              <img v-for="person of customer.people_unknown" class="inline-block h-6 w-6 rounded-full ring-2 ring-white" :src="person.photo" :title="`${person.firstname} ${person.lastname}`" alt="" />
                            </div>
                          </div>
                        </div>
                      </div>
                    </div>
                    <div class="flex items-center">
                      <button type="button" class="py-1 px-3 border bg-gray-100" @click="selectCustomer(customer)">Ver</button>
                      <button type="button" class="py-1 px-3 border bg-gray-100">Editar</button>
                      <button type="button" class="py-1 px-3 border bg-gray-100">Eliminar</button>
                    </div>
                  </div>
                </div>
              </li>
            </ul>
          </div>
        </div>
      </div>
    </main>
    <main v-if="mode == 'profile'">
      <div class="mx-auto max-w-7xl py-3 sm:px-6 lg:px-8">
        <div class="overflow-hidden bg-white shadow sm:rounded-md my-3">
          <ul role="list" class="divide-y divide-gray-200">
            <li>
              <div class="block">
                <div class="flex items-center px-4 py-4 sm:px-6">
                  <div class="flex min-w-0 flex-1 items-center">
                    <div class="flex-shrink-0">
                      <img class="h-12 w-12 rounded-full" :src="customerSelected.photo" alt="" />
                    </div>
                    <div class="min-w-0 flex-1 px-4 md:grid md:grid-cols-3 md:gap-4">
                      <div>
                        <p class="truncate text-sm text-black">{{ customerSelected.firstname }} {{ customerSelected.lastname }}</p>
                        <p class="mt-2 flex items-center text-sm text-black">
                          <EnvelopeIcon class="mr-1.5 h-5 w-5 flex-shrink-0 text-gray-400" aria-hidden="true" />
                          <span class="truncate">{{ customerSelected.email }}</span>
                        </p>
                        <p class="mt-2 flex items-center text-sm text-black">
                          <PhoneIcon class="mr-1.5 h-5 w-5 flex-shrink-0 text-gray-400" aria-hidden="true" />
                          <span class="truncate">{{ customerSelected.cellphone }}</span>
                        </p>
                      </div>
                      <div class="hidden md:block">
                        <div>
                          <p class="text-sm text-gray-900">
                            Birthdate
                            {{ ' ' }}
                            <time :datetime="customerSelected.date">{{ customerSelected.birthdate }}</time>
                          </p>
                          <template v-if="customerSelected.status == 'active'">
                            <p class="mt-2 flex items-center text-sm text-black">
                              <CheckCircleIcon class="mr-1.5 h-5 w-5 flex-shrink-0 text-green-400" aria-hidden="true" />
                              {{ customerSelected.status }}
                            </p>
                          </template>
                          <template v-if="customerSelected.status == 'inactive'">
                            <p class="mt-2 flex items-center text-sm text-black">
                              <MinusCircleIcon class="mr-1.5 h-5 w-5 flex-shrink-0 text-yellow-400" aria-hidden="true" />
                              {{ customerSelected.status }}
                            </p>
                          </template>
                          <template v-if="customerSelected.status == 'blocked'">
                            <p class="mt-2 flex items-center text-sm text-black">
                              <XCircleIcon class="mr-1.5 h-5 w-5 flex-shrink-0 text-red-400" aria-hidden="true" />
                              {{ customerSelected.status }}
                            </p>
                          </template>
                        </div>
                      </div>
                      <div class="hidden md:block">
                        <div>
                          <p class="truncate text-sm text-black">Personas conocidas</p>
                          <div class="flex -space-x-1 overflow-hidden">
                            <img v-for="person of customerSelected.people_known" class="inline-block h-6 w-6 rounded-full ring-2 ring-white" :src="person.photo" :title="`${person.firstname} ${person.lastname}`" alt="" />
                          </div>
                        </div>
                        <div>
                          <p class="truncate text-sm text-black">Personas desconocidas</p>
                          <div class="flex -space-x-1 overflow-hidden">
                            <img v-for="person of customerSelected.people_unknown" class="inline-block h-6 w-6 rounded-full ring-2 ring-white" :src="person.photo" :title="`${person.firstname} ${person.lastname}`" alt="" />
                          </div>
                        </div>
                      </div>
                    </div>
                  </div>
                </div>
              </div>
            </li>
          </ul>
        </div>
      </div>
    </main>
  </div>
</template>

<script setup>
import axios from 'axios'
import { onMounted, ref } from 'vue';
import { Disclosure, DisclosureButton, DisclosurePanel } from '@headlessui/vue'
import { CheckCircleIcon, MinusCircleIcon, XCircleIcon, ChevronRightIcon, EnvelopeIcon, PhoneIcon } from '@heroicons/vue/20/solid'

const navigation = [
  { name: 'Clientes', href: '#', current: true },
]

const stats = ref([
  { name: 'Total', stat: 0 },
])

const formKeyword = ref("")
const customers = ref([])
const customerSelected = ref({})
const mode = ref("list")

onMounted(() => {
  getCustomers()
})

const getCustomers = async () => {
  const response = await axios.get('https://fe-sr-challenge.baymark.net/api/customers', {
    headers: {
      'Authorization': 'Bearer 8def2320e7486ac76d003497fe22ce5a'
    }
  })

  stats.value[0].stat = response.data.total
  customers.value = response.data.data
}

const searchCustomer = async () => {
  let name = encodeURI(formKeyword.value)
  const response = await axios.get(`https://fe-sr-challenge.baymark.net/api/customers?name=${name}`, {
    headers: {
      'Authorization': 'Bearer 8def2320e7486ac76d003497fe22ce5a'
    }
  })

  customers.value = response.data.data
}

const selectCustomer = (customer) => {
  customerSelected.value = customer
  mode.value = 'profile'
}

const unselectCustomer = () => {
  mode.value = 'list'
  customerSelected.value = {}
}
</script>

