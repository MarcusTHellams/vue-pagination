<script setup lang="tsx">
import { api } from '@/common';
import { Edit, Trash2 } from 'lucide-vue-next';
import {
  Button,
  Layout1,
  Pagination,
  PaginationEllipsis,
  PaginationFirst,
  PaginationLast,
  PaginationList,
  PaginationListItem,
  PaginationNext,
  PaginationPrev,
  Select,
  SelectContent,
  SelectItem,
  SelectTrigger,
  SelectValue,
  Table,
  TableBody,
  TableCell,
  TableHead,
  TableHeader,
  TableRow,
  Input,
  Label,
} from '@/components';
import { ProductEditModal } from '@/features';
import { keepPreviousData, useQuery } from '@tanstack/vue-query';
import { FlexRender, getCoreRowModel, useVueTable, type ColumnDef } from '@tanstack/vue-table';
import { ref, computed } from 'vue';

// QUERY PRODUCTS
const page = ref(1);
const limit = ref(5);

interface PaginatedProduct {
  products: Product[];
  total: number;
  skip: number;
  limit: number;
}

interface Product {
  id: number;
  title: string;
  description: string;
  price: number;
  discountPercentage: number;
  rating: number;
  stock: number;
  brand: string;
  category: string;
  thumbnail: string;
  images: string[];
}

const { data } = useQuery<PaginatedProduct, Error>({
  get queryKey() {
    return ['products', page.value, limit.value];
  },
  async queryFn() {
    return api
      .get(`/products?limit=${limit.value}&skip=${(page.value - 1) * limit.value}`)
      .then(({ data }) => data);
  },
  placeholderData: keepPreviousData,
});

const totalPages = computed(() => {
  if (data.value) {
    return Math.ceil(data.value.total / limit.value);
  }
  return 10;
});

const updateLimit = (newLimit: number) => {
  limit.value = newLimit;
  page.value = 1;
};

// END OF QUERY PRODUCTS

// TABLE
const columns: ColumnDef<Product>[] = [
  {
    accessorKey: 'id',
    header: 'Product ID',
  },
  {
    accessorKey: 'title',
    header: 'Title',
  },
  {
    accessorKey: 'description',
    header: 'Description',
  },
  {
    accessorKey: 'price',
    header: 'Price',
    cell(props) {
      return (
        <div class="text-right">
          {new Intl.NumberFormat('en-US', { style: 'currency', currency: 'USD' }).format(
            props.getValue() as number,
          )}
        </div>
      );
    },
  },
  {
    accessorKey: 'rating',
    header: 'Rating',
  },
  {
    accessorKey: 'images',
    header: 'Image',
    cell(props) {
      const images = props.getValue() as string[];
      return (
        <div>
          <img class="object-contain h-20 w-48" src={images[0]} />
        </div>
      );
    },
  },
  {
    header: 'Actions',
    cell() {
      return (
        <div class="flex gap-1">
          {/* @ts-ignore */}
          <Button
            class="p-0 text-blue-500"
            onClick={() => {
              editDialogOpen.value = !editDialogOpen.value;
            }}
            variant={'link'}
          >
            <Edit />
          </Button>
          {/* @ts-ignore */}
          <Button
            class="p-0 text-red-500"
            onClick={() => console.log('Hello World')}
            variant={'link'}
          >
            <Trash2 />
          </Button>
        </div>
      );
    },
  },
];

const table = useVueTable({
  columns,
  getCoreRowModel: getCoreRowModel(),
  get data() {
    return data.value?.products || [];
  },
});

// END OF TABLE

// EDIT DIALOG

const editDialogOpen = ref(false);

// END EDIT DIALOG
</script>

<template>
  <main>
    <Layout1>
      <Table>
        <TableHeader>
          <TableRow v-for="headerGroup in table.getHeaderGroups()" :key="headerGroup.id">
            <TableHead
              v-for="header in headerGroup.headers"
              :key="header.id"
              :colSpan="header.colSpan"
            >
              <FlexRender
                v-if="!header.isPlaceholder"
                :render="header.column.columnDef.header"
                :props="header.getContext()"
              />
            </TableHead>
          </TableRow>
        </TableHeader>
        <TableBody>
          <TableRow v-for="row in table.getRowModel().rows" :key="row.id">
            <TableCell v-for="cell in row.getVisibleCells()" :key="cell.id">
              <FlexRender :render="cell.column.columnDef.cell" :props="cell.getContext()" />
            </TableCell>
          </TableRow>
        </TableBody>
      </Table>
      <p class="text-center">Page {{ page }} of {{ totalPages }}</p>
      <div class="flex items-end gap-3 justify-center">
        <Pagination
          @update:page="(_page) => (page = _page)"
          v-if="data?.products"
          v-slot="{ page }"
          :itemsPerPage="limit"
          :total="data?.total"
          :sibling-count="1"
          show-edges
          :page="page"
        >
          <PaginationList v-slot="{ items }" class="flex items-center gap-1">
            <PaginationFirst />
            <PaginationPrev />
            <template v-for="(item, index) in items">
              <PaginationListItem
                v-if="item.type === 'page'"
                :key="index"
                :value="item.value"
                as-child
              >
                <Button
                  class="w-10 h-10 p-0"
                  :variant="item.value === page ? 'default' : 'outline'"
                >
                  {{ item.value }}
                </Button>
              </PaginationListItem>
              <PaginationEllipsis v-else :key="item.type" :index="index" />
            </template>
            <PaginationNext />
            <PaginationLast />
          </PaginationList>
        </Pagination>
        <div>
          <Label class="mb-1.5" for="page-input">Page</Label>
          <Input
            :min="1"
            :max="totalPages"
            type="number"
            v-model="page"
            name="page-input"
            id="page-input"
          />
        </div>
        <Select
          name="page-limit"
          id="page-limit"
          :modelValue="`${limit}`"
          @update:modelValue="(newValue) => updateLimit(+newValue)"
        >
          <SelectTrigger class="w-1/4">
            <SelectValue />
          </SelectTrigger>
          <SelectContent position="popper" side="top">
            <SelectItem v-for="num in [5, 10, 15, 20, 100]" :key="num" :value="`${num}`">
              {{ num }}
            </SelectItem>
          </SelectContent>
        </Select>
      </div>
    </Layout1>
  </main>
  <ProductEditModal :open="editDialogOpen" />
</template>
