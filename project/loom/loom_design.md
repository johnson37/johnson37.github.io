# Loom Design

## PM

## ELN

### 分子式添加支持别的实验产物

当添加时，通过产物得到chemical_mol, 生成chemical_entry, 添加chemical_entry_follow_links。

两边都需要显示link：分子式这边通过chemical_entry_id, 拿到对应的links，scheme那边，通过product id拿到对应的links

chemical_entry_follow_links:

product_id, chemical_entry_id, soft_del, create_date, modified_date

case 1: 如果分子式删除，删除对应的link  [Done]

case 2: 如果关联产物删除，删除对应的link  [Done]

case3：如果关联scheme删除，删除对应的link [Done]

case4：如果关联scheme所在的实验删除，删除对应的link

```sql
CREATE TABLE app.chemical_entry_follow_link (
    id integer PRIMARY KEY,
    product_id integer NOT NULL REFERENCES app.rxn_product(product_id),
    chemical_entry_id integer NOT NULL REFERENCES app.chemical_drawing_entry(id),
    created_date TIMESTAMP WITHOUT TIME ZONE DEFAULT now(),
    created_by_id INTEGER NOT NULL REFERENCES app.tg_user(user_id),
    soft_del boolean default false
);
```

case 5: Clone Rxn的时候需要把对应的连接clone过去, 连接以及数量都不拿过去。连接默认不会带过来，数量需要清除一下。

case 6: 添加amount 和 unit， 修改分子式的时候，允许修改amount和unit吗？







