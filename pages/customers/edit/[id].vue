<script setup lang="ts">
import { storage } from '#imports';
import { useMutation, useQuery } from '@tanstack/vue-query';
import { v4 as uuid } from 'uuid';
import { DB_ID, COLLECTION_CUSTOMERS, STORAGE_ID } from '#imports';
import type { ICustomer } from '~/types/deals.types';

interface ICustomerFormState
  extends Pick<ICustomer, 'name' | 'email' | 'avatar_url' | 'from_source'> {}

useSeoMeta({
  title: 'Редактирование компании | CRM System',
});

const route = useRoute();
const customerId = route.params.id as string;

const { handleSubmit, defineField, setFieldValue, setValues, values } =
  useForm<ICustomerFormState>();

const { data, isSuccess } = useQuery({
  queryKey: ['get customer', customerId],
  queryFn: () => DB.getDocument(DB_ID, COLLECTION_CUSTOMERS, customerId),
});

watch(isSuccess, () => {
  const initialData = data.value as unknown as ICustomerFormState;
  setValues({
    name: initialData.name,
    email: initialData.email,
    avatar_url: initialData.avatar_url,
    from_source: initialData.from_source || '',
  });
});

const [name, nameAttrs] = defineField('name');
const [email, emailAttrs] = defineField('email');
const [from_source, from_sourceAttrs] = defineField('from_source');

const { mutate, isPending } = useMutation({
  mutationKey: ['update customer', customerId],
  mutationFn: (data: ICustomerFormState) =>
    DB.updateDocument(DB_ID, COLLECTION_CUSTOMERS, customerId, data),
});

const onSubmit = handleSubmit((values) => {
  mutate(values);
});
</script>

<template>
  <div class="p-10">
    <h1 class="font-bold text-2xl mb-10">
      Редактирование {{ (data as unknown as ICustomerFormState)?.name }}
    </h1>
    <form @submit="onSubmit" class="form">
      <UiInput
        placeholder="Наименование"
        v-model="name"
        v-bind="nameAttrs"
        type="text"
        class="input"
      />
      <UiInput
        placeholder="Email"
        v-model="email"
        v-bind="emailAttrs"
        type="text"
        class="input"
      />
      <UiInput
        placeholder="Откуда пришел"
        v-model="from_source"
        v-bind="from_sourceAttrs"
        type="text"
        class="input"
      />

      <UiButton :disabled="isPending" variant="secondary" class="mt-3">{{
        isPending ? 'Загрузка...' : 'Сохранить...'
      }}</UiButton>
    </form>
  </div>
</template>

<style scoped>
.input {
  @apply border-[#161c26] mb-2 placeholder:text-[#748092] focus:border-border transition-colors;
}
</style>
