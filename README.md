# Vinasoy Sale in First Six Months of 2024

## Import libraries and read data

```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns

df = pd.read_csv('First _six_months_data.csv')
```

## Delete some columns that we do not need

```python
df.drop(columns=['STT', 'Miền', 'Vùng', 'Khu vực', 'Số nhà/Khu', 'Tên đường/Xã/Phường', 'TP (Tx)/Quận (Huyện)', 'Trạng thái', 'Hàng bán (Hộp/bịch)', 'Hàng KM (Thùng)', 'Hàng KM (hộp/bịch)', 'Đơn giá lẻ','Chiết khấu (Lì xì)', 'Mã CTKM \n (Mua hàng/Voucher/Phiếu thu hồi)', 'Tên CTKM \n (Mua hàng/Voucher/Phiếu thu hồi)','Số lượng/Điểm trả thưởng \n (Game/Voucher/Loyalty)','Mã chương trình \n (Game/Voucher/Loyalty)',' Tên chương trình \n (Game/Voucher/Loyalty)'], inplace=True)
```

## Show infomation of the dataset

```python
type = df['Loại đơn'].unique()
df = df[df['Loại đơn'] == type[1]]
df.drop(columns=['Loại đơn'], inplace=True)

df['Ngày lấy đơn'] = pd.to_datetime(df['Ngày lấy đơn'], dayfirst=True)
```

## Show sale all months

```python
df['Tháng lấy đơn'] = df['Ngày lấy đơn'].dt.strftime('%B')
df['Tháng'] = df['Ngày lấy đơn'].dt.month

df_sales = df.groupby(['Tháng lấy đơn', 'Tháng'])['Thành tiền'].sum().reset_index()
df_sales = df_sales.sort_values('Tháng', ascending=True)
df_sales.drop(columns=['Tháng'], inplace=True)
```

## Visualization

```python
plt.figure(figsize=(20,6))
sns.lineplot(df_sales,
             x = 'Tháng lấy đơn',
             y = 'Thành tiền',
             marker='o',
             color='#6EB43F'
             )
plt.title('Biểu đồ thể hiện doanh số bán ra 6 tháng đầu năm 2024', fontdict=decor)
for x, y in zip(df_sales['Tháng lấy đơn'], df_sales['Thành tiền']):
    label = "{:,.0f}".format(y)  # Định dạng số thập phân cho label
    plt.annotate(label, (x, y), textcoords="offset points", xytext=(0,2), ha='center', color='#6EB43F', fontsize = 12, fontweight = '600');
```

![alt text](image.png)

# **Visualize product name and name of supermarket systems**
## Extract name of systems from name of customers

```python
df['Hệ thống'] = df['Tên KH'].str.split(' ')
df['Hệ thống'] = df['Hệ thống'].apply(lambda x: x[0])
df['Hệ thống'] = df['Hệ thống'].map({
    'BHX': 'Bách Hóa Xanh',
    'Coopfood': 'Sài Gòn Coop',
    'Coopmart': 'Sài Gòn Coop',
    'Lotte': 'Lotte Mart',
    'VM': 'Vincommerce',
    'VMP': 'Vincommerce',
    'BigC': 'BigC và Go!',
})

df_systems = pd.pivot_table(df, columns='Hệ thống', index='Tên sản phẩm', aggfunc='sum', values='Hàng bán (Thùng)')
```

```python
fig, ax = plt.subplots(len(df_systems.columns), 1, figsize=(20, 10), sharex=True)
# Vòng lặp để vẽ từng biểu đồ cột trên từng subplot
plt.suptitle('Biểu đồ thể hiện sức bán của các sản phẩm theo từng hệ thống siêu thị', color='#6EB43F', fontweight = '600', fontsize = 20)
for i, col in enumerate(df_systems.columns):
    sns.barplot(x=df_systems.index, y=df_systems[col], ax=ax[i], color='#6EB43F')
    ax[i].set_ylabel(col)  # Đặt nhãn trục y cho từng subplot
    ax[i].set_xlabel('Tên sản phẩm')
    ax[i].set_title(f"Biểu đồ thể hiện sức bán của các sản phẩm theo hệ thống siêu thị {col}", color='#6EB43F', fontweight = '600')
# plt.title('Biểu đồ thể hiện sức bán của các sản phẩm theo từng hệ thống siêu thị')
plt.tight_layout()
```

![alt text](image-1.png)

# **Find quality of each order by order ID**

```python
df_qual = df.groupby(['Hệ thống']).agg(
                            Total = ('Hàng bán (Thùng)', 'sum'),
                            Orders = ('Mã đơn hàng', 'nunique')
                            ).reset_index()

df_qual['CLĐH'] = round(df_qual['Total'] / df_qual['Orders'], 2)
df_qual.rename(columns={'Orders': 'Đơn hàng thành công', 'CLĐH': 'Chất lượng đơn hàng'}, inplace=True)

df_qual = df_qual.sort_values('Chất lượng đơn hàng', ascending=False)
plt.figure(figsize=(20,7))
sns.barplot(df_qual, x='Hệ thống', y='Chất lượng đơn hàng', color='#6EB43F', label='Chất lượng đơn hàng')

# Second plot
secondChart = plt.twinx()
sns.lineplot(df_qual, x = 'Hệ thống', y = 'Đơn hàng thành công', marker='o', color='#E9950C', label='Đơn hàng thành công')

plt.title(f'Biểu đồ thể hiện chất lượng đơn hàng và đơn hàng thành công của từng hệ thống siêu thị trong 6 tháng đầu năm 2024', fontdict=decor)
plt.xlabel('Hệ thống siêu thị')
plt.ylim(0, 1200)

plt.legend(loc='upper center')

for x, y in zip(df_qual['Hệ thống'], df_qual['Chất lượng đơn hàng']):
    label = "{:,.2f}".format(y)  # Định dạng số thập phân cho label
    plt.annotate(label, xy=(x, y), textcoords='offset points', xytext=(0,25), ha='center', color='#000', fontsize = 12, fontweight = '600')
    
for t, z in zip(df_qual['Hệ thống'], df_qual['Đơn hàng thành công']):
    label1 = "{:,.0f}".format(z)  # Định dạng số thập phân cho label
    plt.annotate(label1, xy=(t, z), textcoords='offset points', xytext=(0,5), ha='center', color='#000', fontsize = 10, fontweight = '600')
```

![alt text](image-2.png)

# **Find quantity of orders and quality of each ordereach month**

```python
orders = df.groupby(['Tháng lấy đơn', 'Tháng']).agg({'Hàng bán (Thùng)': 'sum', 'Mã đơn hàng': 'nunique'}).reset_index()
orders = orders.sort_values('Tháng', ascending=True)
orders = orders.rename(columns={'Hàng bán (Thùng)': 'Sản lượng', 'Mã đơn hàng': 'Đơn hàng thành công'})
orders.drop(columns='Tháng', inplace=True)
orders['Chất lượng đơn hàng'] = round(orders['Sản lượng'] / orders['Đơn hàng thành công'], 2)

plt.figure(figsize=(20,7))
sns.lineplot(orders, x='Tháng lấy đơn', y='Đơn hàng thành công', marker='o', color='#6EB43F', label='Đơn hàng thành công', linewidth='3')
plt.ylabel('Đơn hàng thành công (Đơn)')
plt.ylim((0,500))
plt.twinx()
sns.lineplot(orders, x='Tháng lấy đơn', y='Chất lượng đơn hàng', marker='o', color='#E9950C', label='Chất lượng đơn hàng', linewidth='3')
plt.ylabel('Chất lượng đơn hàng (Thùng/Đơn)')
plt.legend(loc='upper center')
plt.grid()
plt.title('Biểu đồ thể hiện đơn hàng thành công và chất lượng đơn hàng theo từng tháng của toàn kênh MT - Đông Nam Bộ 1', color='#6EB43F', fontweight = '600')
for x, y in zip(orders['Tháng lấy đơn'], orders['Đơn hàng thành công']):
    label = "{:,.2f}".format(y)  # Định dạng số thập phân cho label
    plt.annotate(label, xy=(x, y), textcoords='offset points', xytext=(0,5), ha='center', color='#000', fontsize = 12, fontweight = '600')
    
for t, z in zip(orders['Tháng lấy đơn'], orders['Chất lượng đơn hàng']):
    label1 = "{:,.0f}".format(z)  # Định dạng số thập phân cho label
    plt.annotate(label1, xy=(t, z), textcoords='offset points', xytext=(0,5), ha='center', color='#000', fontsize = 10, fontweight = '600')
```

![alt text](image-3.png)

# **Visualize sales of all contrbutors in team**

## Create pivot table

```python
contributors = pd.pivot_table(df.sort_values('Tháng', ascending=True),index=['Tháng','Tháng lấy đơn'], columns='Tên NPP', values='Thành tiền', aggfunc='sum')
```

## Visualization

```python
import matplotlib.ticker as ticker
plt.figure(figsize=(20,7))
for contributor in contributors.columns:
    sns.lineplot(contributors, x='Tháng lấy đơn', y=contributor, label=contributor, linewidth=3, marker='o')
    plt.legend(loc='upper right')
    plt.ylabel('')
plt.grid()
formatter = ticker.FuncFormatter(lambda x, pos: '{:,.0f}'.format(x))
plt.gca().yaxis.set_major_formatter(formatter)
plt.title('Biểu đồ thể hiện sản lượng bán ra của các nhà phân phối theo từng tháng của toàn kênh MT - Đông Nam Bộ 1', color='#6EB43F', fontweight = '600');
```

![alt text](image-4.png)

# **Analyze sales of MT systems in each contributors**
## Create pivot table

```python
systems = pd.pivot_table(df.sort_values('Tháng', ascending=True), index=['Hệ thống'], columns=['Tên NPP'], values='Thành tiền', aggfunc='sum')
```

## Visualization
```python
# Tạo subplot
fig, axs = plt.subplots(2, 2, figsize=(20, 7))

# Flat list của các subplot axes
axes = axs.flatten()
formatter = ticker.FuncFormatter(lambda x, pos: '{:,.0f}'.format(x))

# Vẽ biểu đồ thanh cho mỗi NPP
for ax, npp in zip(axes, systems.columns):  # Bỏ qua cột 'Hệ thống'
    sns.barplot(data=systems.sort_values('Hệ thống', ascending=True), x='Hệ thống', y=npp, ax=ax)
    ax.set_title(f'Biểu đồ thể hiện sản lượng của các hệ thống siêu thị tại nhà phân phối {npp}')  # Đặt tiêu đề cho từng subplot
    ax.set_xticklabels(ax.get_xticklabels(), rotation=45, ha='right')  # Xoay nhãn trục x
    ax.yaxis.set_major_formatter(formatter)

# Điều chỉnh layout
plt.tight_layout();
```

![alt text](image-5.png)