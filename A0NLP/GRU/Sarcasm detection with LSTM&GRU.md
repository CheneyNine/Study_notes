# 1. 加载和预处理数据集
```python
# 读取数据集

df = pd.read_csv('https://drive.google.com/uc?id=1D3Ma2HcEkkt85d5OtXxL5f8MHmrwvuQi&export=download', sep=',')

# Keep only those rows of the data frames with score>10

# 初步筛选

df_upten = df[df['score'] > 10]

# Make a copy of the dataframe

df_copy = df_upten[['label', 'comment']].copy()

# 筛选训练集和测试集
length = len(df_copy)

temp = int(length * 0.8)

data_train = df_copy[:temp]

data_test = df_copy[temp:]

```




# 2. Character level tokenization and filtering of smaller/longer sentences

* Tokenize the data by fitting the train data with a vocabulary size of 100, using character-level tokenization instead of whitespace tokenization. 词汇大小为100，字符用标记而不是空白

* Preprocess the data, so that the label column is of type int and the comments are represented as vectors got by tokenization.

* Filter tokenized comments that have less than 10 and more than 100 tokens for both the train and test sets.


```python
# Define a tokenizer with vocabulary size of 100 that filters all special characters

data_train.comment = data_train.comment.str.replace("[^a-zA-Z0-9]", '', regex=True).str.lower()

data_test.comment = data_test.comment.str.replace("[^a-zA-Z0-9]", '', regex=True).str.lower()

# 定义自己的 tokenizer

tokenizer = tf.keras.preprocessing.text.Tokenizer(num_words=100, char_level=True, oov_token='UNK')

# Fit the tokenizer on the comments column of the train set

tokenizer.fit_on_texts(data_train['comment'])
word_index = tokenizer.word_index

# Convert the label column of the train and test
label_train = data_train.label.values
label_test = data_test.label.values
# Convert the comments column to sequences
data_train = tokenizer.texts_to_sequences(data_train['comment'])
data_test = tokenizer.texts_to_sequences(data_test['comment'])

# Create a boolean mask to filter out sentences which are less than 10 tokens/ more than 100 tokens.
mask_train = [(10 <= len(seq) <= 100) for seq in data_train]
data_train = [seq for seq, mask in zip(data_train, mask_train) if mask]
label_train = [seq for seq, mask in zip(label_train, mask_train) if mask]

# Similarly create a mask to filter the test dataset
mask_test = [(10 <= len(seq) <= 100) for seq in data_test]
data_test= [seq for seq, mask in zip(data_test, mask_test) if mask]
label_test = [seq for seq, mask in zip(label_test, mask_train) if mask]
```

重要的几个函数：

1. 定义自己的 tokenizer

```python
tokenizer = tf.keras.preprocessing.text.Tokenizer(num_words=100, char_level=True, oov_token='UNK')
```

2. 生成 tokenizer 并应用

```python
tokenizer.fit_on_texts(data_train['comment'])
data_train = tokenizer.texts_to_sequences(data_train['comment'])
data_test = tokenizer.texts_to_sequences(data_test['comment'])
```


# 3. Tensorflow Dataset制作（training & validation)

1. 数据集转换成层次不齐（ragged）的张量
2. 创建张量流数据集
3. 设置批次，对数据集进行混洗（shuffle）和批量（batch）处理
4. 用 map 函数填充数据
5. 🌟预取（prefetch）操作，对数据提前加载减少网络等待的时间

```python
# Use tensorflow ragged constants to get the ragged version of the train data
ragged_train = tf.ragged.constant(data_train)
label_train = tf.ragged.constant(label_train)

# Convert the train set into a tensorflow dataset
dataset_train = tf.data.Dataset.from_tensor_slices((ragged_train, label_train))

# Shuffle the dataset
buffer_size = len(data_train)
dataset_train = dataset_train.shuffle(buffer_size)


# Batch the dataset into batches of 512
batch_size = 512
dataset_train = dataset_train.batch(batch_size)

# Pad each input batch with 0s
dataset_train = dataset_train.map(lambda x,y: (x.to_tensor(default_value=0, shape=[None, None]), y), num_parallel_calls=5)

# Prefetching
prefetch_buffer_size = tf.data.experimental.AUTOTUNE
dataset_train = dataset_train.prefetch(prefetch_buffer_size)

# Use tensorflow ragged constants to get the ragged version of the test data
ragged_test_data = tf.ragged.constant(data_test)

# Index to word mapping
index_2_word = {index: word for word, index in tokenizer.word_index.items()}


# Reconstruct the sentence using tokens
''.join([index_2_word[i] for i in X_tr[0].numpy() if i!=0])

# Checking sequence length counts across batches
seq_len = []
for X_tr, y_tr in dataset_train.as_numpy_iterator():
seq_len.append(X_tr.shape[1])

# Checking the count of seq lengths
Counter(seq_len)


```

# 4. 模型定义和训练

构建 GRU 模型
```python
# Define the model

model2 = Sequential()
model2.add(Embedding(input_dim=10000, output_dim=128))
# CNN layers
model2.add(Conv1D(filters=64, kernel_size=5, activation='relu'))
model2.add(MaxPooling1D(pool_size=4))
# Deep Bidirectional GRU layers
# Assuming you already have a GRU setup, add it here
model2.add(Bidirectional(GRU(64, return_sequences=True)))
model2.add(Bidirectional(GRU(64)))
# Dense layers for final processing and output
# Tailor this to your specific task (e.g., number of units and activation function)
model2.add(Dense(64, activation='relu'))
model2.add(Dropout(0.5))
model2.add(Dense(1, activation='sigmoid')) # Adjust for your specific task

# Compile the model
model2.compile(optimizer='adam', loss='binary_crossentropy', metrics=['accuracy'])
# Model summary
model2.summary()
```

# 5. Evaluate the model模型评估