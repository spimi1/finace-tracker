<template>
  <UModal v-model="isOpen">
    <UCard>
      <template #header> Add Transaction </template>
      <UForm :state="state" :schema="schema" ref="form">
        <!-- Transakcje -->
        <UFormGroup
          :required="true"
          label="Transaction Type"
          name="type"
          class="mb-4"
        >
          <USelect
            placeholder="Select the transaction type"
            :options="types"
            icon="i-line-md:check-list-3-twotone"
            v-model="state.type"
          />
        </UFormGroup>

        <!-- Kwota -->
        <UFormGroup label="Amount" :required="true" name="amount" class="mb-4">
          <UInput
            tpye="number"
            placeholder="Amount"
            icon="i-line-md:briefcase"
            v-model.number="state.amount"
          />
        </UFormGroup>

        <!-- Data -->
        <UFormGroup
          label="Transaction date"
          :required="true"
          name="created_at"
          class="mb-4"
        >
          <UInput
            type="date"
            icon="i-line-md:calendar"
            v-model="state.created_at"
          />
        </UFormGroup>

        <!-- Opis -->
        <UFormGroup
          label="Description"
          hint="Optional"
          name="description"
          class="mb-4"
        >
          <UInput
            placeholder="Description"
            icon="i-line-md:align-left"
            v-model="state.description"
          />
        </UFormGroup>

        <!-- Kategoria -->
        <UFormGroup
          :required="true"
          label="Category"
          name="category"
          class="mb-4"
          v-if="state.type == 'Expense'"
        >
          <USelect
            placeholder="Category"
            :options="categories"
            icon="i-line-md:folder"
            v-model="state.category"
          />
        </UFormGroup>

        <UButton
          type="button"
          color="black"
          variant="solid"
          label="Save"
          :loading="isLoading"
          @click="save"
        />
      </UForm>
    </UCard>
  </UModal>
</template>

<script setup>
import { categories, types } from "~/constants";
import { z } from "zod";

const props = defineProps({
  modelValue: Boolean,
});
const emit = defineEmits(["update:modelValue", "saved"]);

const defaultSchema = z.object({
  created_at: z.string(),
  description: z.string().optional(),
  amount: z.number().positive("Amount needs to be more than 0"),
});

const incomeSchema = z.object({
  type: z.literal("Income"),
});
const expenseSchema = z.object({
  type: z.literal("Expense"),
  category: z.enum(categories),
});
const investmentSchema = z.object({
  type: z.literal("Investment"),
});
const savingSchema = z.object({
  type: z.literal("Saving"),
});

const schema = z.intersection(
  z.discriminatedUnion("type", [
    incomeSchema,
    expenseSchema,
    investmentSchema,
    savingSchema,
  ]),
  defaultSchema
);

const form = ref();
const isLoading = ref(false);
const supabase = useSupabaseClient();
const toast = useToast();

const save = async () => {
  if (form.value.errors.length) return;

  isLoading.value = true;
  try {
    const { error } = await supabase
      .from("transactions")
      .upsert({ ...state.value });

    if (!error) {
      toast.add({
        title: "Transaction saved",
        icon: "i-line-md:circle-to-confirm-circle-twotone-transition",
      });
      isOpen.value = false;
      emit("saved");
      return;
    }

    throw error;
  } catch (e) {
    toast.add({
      title: "Transaction not saved",
      description: e.message,
      icon: "i-line-md:close-circle-twotone",
      color: "red",
    });
  } finally {
    isLoading.value = false;
  }
};

const initialState = {
  type: undefined,
  amount: 0,
  created_at: undefined,
  description: undefined,
  category: undefined,
};
const state = ref({
  ...initialState,
});
const resetForm = () => {
  Object.assign(state.value, initialState);
  form.value.clear();
};

const isOpen = computed({
  get: () => props.modelValue,
  set: (value) => {
    if (!value) resetForm();
    emit("update:modelValue", value);
  },
});
</script>
