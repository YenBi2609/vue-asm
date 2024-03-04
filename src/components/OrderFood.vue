<template>
  <v-stepper
    :items="['Step 1', 'Step 2', 'Step 3', 'Step 4']"
    class="pa-6"
    hide-actions
    v-model="step"
  >
    <template v-slot:item.1>
      <v-card class="config-step" flat>
        <div>Please Select a meal</div>
        <v-select
          density="compact"
          :rules="[rules.require]"
          :items="['breakfast', 'lunch', 'dinner']"
          v-model="order.meal"
        ></v-select>

        <div>Please Enter Number of people</div>
        <v-text-field
          density="compact"
          single-line
          :rules="[rules.require, rules.maxNumberOfPeople]"
          type="number"
          v-model="order.numberOfPeople"
        />
        <v-btn
          color="primary"
          @click="step++"
          style="float: right"
          :disabled="!(order.meal && order.numberOfPeople <= 10)"
        >
          Next
        </v-btn>
      </v-card>
    </template>

    <template v-slot:item.2>
      <v-card class="config-step" flat>
        <div>Please Select a Restaurant</div>
        <v-select
          density="compact"
          :rules="[rules.require]"
          :items="listRestaurant"
          item-title="restaurant"
          v-model="order.restaurant"
        ></v-select>
        <div class="d-flex justify-space-between">
          <v-btn color="gray" @click="step--"> Previous </v-btn>
          <v-btn color="primary" @click="step++" :disabled="!order.restaurant">
            Next
          </v-btn>
        </div>
      </v-card>
    </template>

    <template v-slot:item.3>
      <v-card class="config-step" flat>
        <div class="d-flex">
          <div>Please Select a Dish</div>
          <div>Please enter no. of servings</div>
        </div>
        <div
          class="d-flex"
          v-for="(dishes, index) in order.listDishesSelected"
          :key="index"
        >
          <v-select
            density="compact"
            :rules="[rules.require]"
            :items="listDishes"
            item-title="name"
            class="mr-4"
            v-model="dishes.name"
            @update:model-value="handleAfterSelectDishes(dishes.name, index)"
          ></v-select>

          <v-text-field
            density="compact"
            single-line
            :rules="[rules.require]"
            type="number"
            v-model="dishes.quantity"
          />
        </div>
        <v-btn
          icon="mdi-plus-circle"
          class="mb-4"
          @click="handleAddDishes"
        ></v-btn>
        <div class="d-flex justify-space-between">
          <v-btn color="gray" @click="step--"> Previous </v-btn>

          <v-btn color="primary" @click="handleCheckDishes"> Next </v-btn>
        </div>
      </v-card>
    </template>

    <template v-slot:item.4>
      <v-card class="config-step" flat>
        <div class="d-flex justify-space-between mb-6">
          <div>Meal</div>
          <div>{{ order.meal }}</div>
        </div>
        <div class="d-flex justify-space-between mb-6">
          <div>No. of. People</div>
          <div>{{ order.numberOfPeople }}</div>
        </div>
        <div class="d-flex justify-space-between mb-6">
          <div>Restaurant</div>
          <div>{{ order.restaurant }}</div>
        </div>
        <div class="d-flex mb-4 justify-space-between mb-6">
          <div>Dishes</div>
          <v-table>
            <thead>
              <tr>
                <th class="text-left">Dish</th>
                <th class="text-left">No. of servings</th>
              </tr>
            </thead>
            <tbody>
              <tr
                v-for="(dishes, index) in order.listDishesSelected"
                :key="index"
              >
                <td>{{ dishes.name }}</td>
                <td>{{ dishes.quantity }}</td>
              </tr>
            </tbody>
          </v-table>
        </div>
        <div class="d-flex justify-space-between">
          <v-btn color="gray" @click="step--"> Previous </v-btn>

          <v-btn color="primary" @click="submitOrder"> Submit </v-btn>
        </div>
      </v-card>
    </template>
  </v-stepper>
  <v-snackbar v-model="showNotification" multi-line :timeout="5000">
    {{ notificationContent }}

    <template v-slot:actions>
      <v-btn color="red" variant="text" @click="showNotification = false">
        Close
      </v-btn>
    </template>
  </v-snackbar>
</template> 

<script setup lang="ts">
import { computed, ref } from "vue";
import data from "../data/dishes.json";

const step = ref(1);
const showNotification = ref(false);
const notificationContent = ref("");

const rules = ref({
  require: (value: any) => !!value || "Required.",
  maxNumberOfPeople: (value: any) =>
    (value && value <= 10) || "Maximum 10 people",
});

const order = ref({
  meal: "",
  numberOfPeople: 1,
  restaurant: "",
  listDishesSelected: [
    {
      name: "",
      quantity: 1,
    },
  ],
});

const listRestaurant = computed(() => {
  return data.dishes.filter((d) => d.availableMeals.includes(order.value.meal));
});

const listDishes = computed(() => {
  return data.dishes.filter(
    (dishes) =>
      dishes.restaurant == order.value.restaurant &&
      dishes.availableMeals.includes(order.value.meal)
  );
});

const handleAddDishes = () => {
  if (order.value.listDishesSelected.length == 10) {
    showNotification.value = true;
    notificationContent.value = "You can only choose a maximum of 10 dishes";
    return;
  }
  order.value.listDishesSelected.push({
    name: "",
    quantity: 1,
  });
};

const handleAfterSelectDishes = (dishName: string, index: number) => {
  const dishes = order.value.listDishesSelected.filter(
    (dishes, i) => dishes.name == dishName && i != index
  );
  if (dishes.length > 0) {
    showNotification.value = true;
    notificationContent.value =
      "The dish has been selected, please add more servings";
    order.value.listDishesSelected[index].name = "";
  }
};

const handleCheckDishes = () => {
  let sum = 0;
  order.value.listDishesSelected.forEach((dishes) => {
    sum += dishes.quantity;
  });
  if (sum < order.value.numberOfPeople) {
    showNotification.value = true;
    notificationContent.value = `Not enough food for ${order.value.numberOfPeople} people`;
    return;
  }
  step.value++;
};

const submitOrder = () => {
  console.log(order.value);
};
</script>

<style scoped>
.config-step {
  margin: auto;
  width: 60%;
  padding: 8px;
}
</style>
