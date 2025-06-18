```
<AutoGridLayout :cols="{ default: 1, sm: 2, md: 3, lg: 4, xl: 5, '2xl': 6, '3xl': 6, '4xl': 8, '5xl': 12 }" gap="16px" class="mt-6 overflow-hidden overflow-y-auto modelList">
   <div v-for="item in tableData" class="item-inner"> </div>
</AutoGridLayout>



.item-inner {
  width: 100%;
  aspect-ratio: 14 / 15; /* 只用来算高度 */
}
```
