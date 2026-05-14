# Deep Learning Cheat Sheet (Keras / TensorFlow)

TensorFlow and Keras are the dominant production-grade tools for deep learning in Python. This cheat sheet covers the API patterns you'll reach for most — model building, training, regularization, saving, and data pipelines. Each entry explains when to use the pattern and shows the minimal working code.

---

## 1. Building Models

### Sequential API

Use when layers connect one-to-one in a straight line. Fastest way to prototype feedforward and simple CNN/RNN architectures.

```python
import tensorflow as tf
from tensorflow import keras

model = keras.Sequential([
    keras.layers.Dense(128, activation='relu', input_shape=(20,)),
    keras.layers.Dense(64, activation='relu'),
    keras.layers.Dense(1, activation='sigmoid'),
])

model.summary()
# Output:
# Model: "sequential"
# _________________________________________________________________
#  Layer (type)              Output Shape         Param #
# =================================================================
#  dense (Dense)             (None, 128)          2688
#  dense_1 (Dense)           (None, 64)           8256
#  dense_2 (Dense)           (None, 1)            65
# =================================================================
# Total params: 11,009
```

### Functional API

Use when you need multiple inputs, multiple outputs, skip connections, or shared layers. The Functional API makes the graph explicit.

```python
import tensorflow as tf
from tensorflow import keras

# Two-input model: tabular features + an ID embedding
tabular_input = keras.Input(shape=(20,), name='tabular')
id_input = keras.Input(shape=(1,), name='user_id')

# Branch 1
x = keras.layers.Dense(64, activation='relu')(tabular_input)

# Branch 2
embedded = keras.layers.Embedding(input_dim=1000, output_dim=8)(id_input)
embedded = keras.layers.Flatten()(embedded)

# Merge
merged = keras.layers.Concatenate()([x, embedded])
output = keras.layers.Dense(1, activation='sigmoid', name='prediction')(merged)

model = keras.Model(inputs=[tabular_input, id_input], outputs=output)
model.summary()
```

### Subclassing (Custom Model)

Use when you need dynamic computation graphs, non-standard forward passes, or research-level flexibility.

```python
import tensorflow as tf
from tensorflow import keras

class ResidualBlock(keras.Model):
    def __init__(self, units):
        super().__init__()
        self.dense1 = keras.layers.Dense(units, activation='relu')
        self.dense2 = keras.layers.Dense(units)
        self.activation = keras.layers.Activation('relu')

    def call(self, inputs, training=False):
        x = self.dense1(inputs)
        x = self.dense2(x)
        return self.activation(x + inputs)  # skip connection

block = ResidualBlock(units=32)
out = block(tf.ones((4, 32)))
print(out.shape)  # Output: (4, 32)
```

---

## 2. Common Layers

### Dense

The standard fully connected layer. Every input neuron connects to every output neuron. Workhorse of tabular and final classification heads.

```python
from tensorflow import keras

# units=64 outputs, relu activation, L2 regularization
layer = keras.layers.Dense(
    units=64,
    activation='relu',
    kernel_regularizer=keras.regularizers.l2(1e-4),
)
```

### Dropout

Randomly zeros a fraction of activations during training to prevent co-adaptation of neurons. Only active during training — Keras handles this automatically when you pass `training=True` (done internally by `model.fit`).

```python
from tensorflow import keras

model = keras.Sequential([
    keras.layers.Dense(256, activation='relu'),
    keras.layers.Dropout(rate=0.4),   # drop 40% of units each forward pass
    keras.layers.Dense(10, activation='softmax'),
])
```

### BatchNormalization

Normalizes activations across the mini-batch at each layer. Reduces internal covariate shift, allows higher learning rates, and often replaces the need for Dropout in CNNs. Place it after the linear transform, before the activation.

```python
from tensorflow import keras

model = keras.Sequential([
    keras.layers.Dense(128),
    keras.layers.BatchNormalization(),   # normalize, then activate
    keras.layers.Activation('relu'),
    keras.layers.Dense(10, activation='softmax'),
])
```

### Conv2D and MaxPooling2D

`Conv2D` learns spatial feature detectors (edges, textures, patterns). `MaxPooling2D` downsamples the spatial dimensions, reducing compute and giving translation invariance.

```python
from tensorflow import keras

model = keras.Sequential([
    # 32 filters, 3x3 kernel, 'same' padding preserves spatial size
    keras.layers.Conv2D(32, kernel_size=(3, 3), activation='relu',
                        padding='same', input_shape=(28, 28, 1)),
    keras.layers.MaxPooling2D(pool_size=(2, 2)),  # halves spatial dims
    keras.layers.Conv2D(64, kernel_size=(3, 3), activation='relu', padding='same'),
    keras.layers.MaxPooling2D(pool_size=(2, 2)),
    keras.layers.Flatten(),
    keras.layers.Dense(10, activation='softmax'),
])
```

### LSTM

Long Short-Term Memory processes sequences while retaining long-range dependencies. Use for time series, text, or any ordered data where context from earlier steps matters.

```python
from tensorflow import keras

model = keras.Sequential([
    # return_sequences=True passes the full output at each step to next layer
    keras.layers.LSTM(64, return_sequences=True, input_shape=(30, 10)),
    keras.layers.LSTM(32),   # final LSTM — returns only the last time step
    keras.layers.Dense(1),
])
# input_shape=(timesteps=30, features=10)
```

### Embedding

Converts integer token IDs into dense vectors. Essential for NLP — it learns a semantic vector space where similar words cluster together.

```python
from tensorflow import keras

# vocab_size=10000 possible tokens, embedding dim=64
embedding_layer = keras.layers.Embedding(input_dim=10000, output_dim=64)

# Input: batch of integer sequences shape (batch, sequence_length)
import tensorflow as tf
sample_input = tf.constant([[1, 5, 200, 999], [4, 23, 1, 0]])
output = embedding_layer(sample_input)
print(output.shape)  # Output: (2, 4, 64)
```

---

## 3. Activation Functions

### relu (Rectified Linear Unit)

Default choice for hidden layers in feedforward and convolutional networks. Computationally cheap, avoids the vanishing gradient problem that plagues sigmoid/tanh in deep networks.

```python
from tensorflow import keras

layer = keras.layers.Dense(64, activation='relu')
# Equivalent:
layer = keras.layers.Dense(64)
act = keras.layers.Activation('relu')
```

### sigmoid

Squashes output to (0, 1). Use only in the final layer for binary classification. Suffers from vanishing gradients in deep hidden layers — do not use it there.

```python
from tensorflow import keras

output_layer = keras.layers.Dense(1, activation='sigmoid')
# Pair with loss='binary_crossentropy'
```

### softmax

Converts a vector of logits into a probability distribution (sums to 1). Use in the final layer for multi-class classification.

```python
from tensorflow import keras

output_layer = keras.layers.Dense(10, activation='softmax')
# Pair with loss='categorical_crossentropy' (one-hot labels)
# or loss='sparse_categorical_crossentropy' (integer labels)
```

### tanh

Squashes to (−1, 1). Zero-centered, which can help gradient flow compared to sigmoid. Used in LSTM/GRU gates internally. Sometimes better than sigmoid for hidden layers in shallow nets.

```python
from tensorflow import keras

layer = keras.layers.Dense(64, activation='tanh')
```

### LeakyReLU

Fixes the "dying ReLU" problem by allowing a small negative slope instead of zeroing negative inputs. Use when you see dead neurons (weights that never update) with standard relu.

```python
from tensorflow import keras

model = keras.Sequential([
    keras.layers.Dense(128),
    keras.layers.LeakyReLU(negative_slope=0.1),  # 10% of negative signal passes
    keras.layers.Dense(10, activation='softmax'),
])
```

---

## 4. Compiling

### model.compile() — The Core Pattern

Compiling wires together the loss function, optimizer, and evaluation metrics. You must call this before `fit()`.

```python
model.compile(
    optimizer='adam',
    loss='sparse_categorical_crossentropy',  # integer labels
    metrics=['accuracy'],
)
```

### Loss Function Selection

Choose based on your task type. Wrong loss is the most common silent failure in deep learning.

```python
from tensorflow import keras

# Binary classification (single sigmoid output)
model.compile(optimizer='adam', loss='binary_crossentropy', metrics=['accuracy'])

# Multi-class, one-hot encoded labels
model.compile(optimizer='adam', loss='categorical_crossentropy', metrics=['accuracy'])

# Multi-class, integer labels (more memory efficient)
model.compile(optimizer='adam', loss='sparse_categorical_crossentropy', metrics=['accuracy'])

# Regression
model.compile(optimizer='adam', loss='mse', metrics=['mae'])

# Regression when outliers are present (less sensitive than mse)
model.compile(optimizer='adam', loss='huber', metrics=['mae'])
```

### Optimizer Configuration

Pass a string for defaults, or an optimizer object to control hyperparameters.

```python
from tensorflow import keras

# String shorthand (uses default lr)
model.compile(optimizer='adam', loss='mse')

# Explicit object — control learning rate and other params
adam = keras.optimizers.Adam(learning_rate=1e-3, beta_1=0.9, beta_2=0.999)
sgd = keras.optimizers.SGD(learning_rate=0.01, momentum=0.9, nesterov=True)

model.compile(optimizer=adam, loss='sparse_categorical_crossentropy', metrics=['accuracy'])
```

### Multiple Metrics

Track more than one metric at compile time — they all appear in the training history.

```python
model.compile(
    optimizer='adam',
    loss='binary_crossentropy',
    metrics=[
        'accuracy',
        keras.metrics.AUC(name='auc'),
        keras.metrics.Precision(name='precision'),
        keras.metrics.Recall(name='recall'),
    ],
)
```

---

## 5. Training

### model.fit() — Basic Usage

`batch_size` controls how many samples update the weights at once. `epochs` is the number of full passes through the training data. The returned `history` object holds loss and metric curves.

```python
history = model.fit(
    X_train, y_train,
    batch_size=32,
    epochs=50,
    validation_split=0.2,   # holds out last 20% of training data for validation
    verbose=1,
)

# Access training curves
import pandas as pd
hist_df = pd.DataFrame(history.history)
print(hist_df[['loss', 'val_loss']].tail())
```

### validation_split vs validation_data

`validation_split` is convenient but splits from the end of the array (not shuffled). `validation_data` gives you control — use it when you have a proper held-out set.

```python
# validation_split — fast, but uses the last fraction of X_train as-is
history = model.fit(X_train, y_train, epochs=20, validation_split=0.15)

# validation_data — explicit, preferred for time series or when you've done your own split
history = model.fit(
    X_train, y_train,
    epochs=20,
    validation_data=(X_val, y_val),
)
```

### class_weight — Handling Imbalanced Datasets

When one class dominates, the model learns to predict it always. `class_weight` penalizes errors on minority classes more heavily.

```python
from sklearn.utils.class_weight import compute_class_weight
import numpy as np

classes = np.unique(y_train)
weights = compute_class_weight('balanced', classes=classes, y=y_train)
class_weight_dict = dict(zip(classes, weights))

history = model.fit(
    X_train, y_train,
    epochs=30,
    class_weight=class_weight_dict,
)
```

### model.predict() and model.evaluate()

```python
# Evaluate on test set — returns [loss, metric1, metric2, ...]
test_loss, test_acc = model.evaluate(X_test, y_test, verbose=0)
print(f'Test accuracy: {test_acc:.4f}')

# Predict probabilities
probs = model.predict(X_test)          # shape: (n_samples, n_classes)
labels = probs.argmax(axis=1)          # for multi-class
```

---

## 6. Callbacks

### EarlyStopping

Stops training when the monitored metric stops improving. `patience` is how many epochs to wait before stopping. `restore_best_weights=True` rolls back to the best checkpoint automatically.

```python
from tensorflow import keras

early_stop = keras.callbacks.EarlyStopping(
    monitor='val_loss',
    patience=10,
    restore_best_weights=True,
    verbose=1,
)

model.fit(X_train, y_train, epochs=200, validation_split=0.2,
          callbacks=[early_stop])
```

### ModelCheckpoint

Saves the model (or just weights) to disk whenever the monitored metric improves. Use this alongside EarlyStopping so you always have the best model persisted.

```python
from tensorflow import keras

checkpoint = keras.callbacks.ModelCheckpoint(
    filepath='best_model.keras',
    monitor='val_loss',
    save_best_only=True,
    verbose=1,
)

model.fit(X_train, y_train, epochs=100, validation_split=0.2,
          callbacks=[checkpoint])
```

### ReduceLROnPlateau

Cuts the learning rate by a factor when the loss stops improving. Useful when training plateaus — the smaller step size can navigate out of a flat region.

```python
from tensorflow import keras

reduce_lr = keras.callbacks.ReduceLROnPlateau(
    monitor='val_loss',
    factor=0.5,         # new_lr = lr * factor
    patience=5,
    min_lr=1e-6,
    verbose=1,
)

model.fit(X_train, y_train, epochs=100, validation_split=0.2,
          callbacks=[reduce_lr])
```

### TensorBoard

Logs training metrics, histograms, and computation graphs to a directory. Launch with `tensorboard --logdir logs/` from the terminal. Essential for debugging training dynamics.

```python
from tensorflow import keras
import datetime

log_dir = 'logs/fit/' + datetime.datetime.now().strftime('%Y%m%d-%H%M%S')
tensorboard_cb = keras.callbacks.TensorBoard(
    log_dir=log_dir,
    histogram_freq=1,   # log weight histograms every epoch
)

model.fit(X_train, y_train, epochs=50, validation_split=0.2,
          callbacks=[tensorboard_cb])
# Run: tensorboard --logdir logs/fit/
```

### Combining Callbacks

Pass a list — all callbacks run together.

```python
from tensorflow import keras

callbacks = [
    keras.callbacks.EarlyStopping(monitor='val_loss', patience=10,
                                  restore_best_weights=True),
    keras.callbacks.ModelCheckpoint('best_model.keras', save_best_only=True),
    keras.callbacks.ReduceLROnPlateau(monitor='val_loss', factor=0.5, patience=5),
]

model.fit(X_train, y_train, epochs=200, validation_split=0.2, callbacks=callbacks)
```

---

## 7. Regularization

### Dropout

Randomly deactivates neurons during each training forward pass. Forces the network to learn redundant representations. Rate between 0.2–0.5 is typical. Never apply it to the output layer.

```python
from tensorflow import keras

model = keras.Sequential([
    keras.layers.Dense(512, activation='relu', input_shape=(100,)),
    keras.layers.Dropout(0.3),
    keras.layers.Dense(256, activation='relu'),
    keras.layers.Dropout(0.3),
    keras.layers.Dense(1, activation='sigmoid'),
])
```

### L1 and L2 Kernel Regularization

L2 (weight decay) penalizes large weights and is the most common form. L1 drives some weights to exactly zero (sparse models). Apply via `kernel_regularizer` in any layer.

```python
from tensorflow import keras

l2_layer = keras.layers.Dense(
    128, activation='relu',
    kernel_regularizer=keras.regularizers.l2(1e-4),   # lambda = 0.0001
)

l1_layer = keras.layers.Dense(
    64, activation='relu',
    kernel_regularizer=keras.regularizers.l1(1e-4),
)

l1_l2_layer = keras.layers.Dense(
    64, activation='relu',
    kernel_regularizer=keras.regularizers.l1_l2(l1=1e-5, l2=1e-4),
)
```

### BatchNormalization as Regularizer

BatchNorm has a mild regularizing effect by adding noise through batch statistics. In CNNs, it often replaces Dropout. It also lets you use higher learning rates, which speeds convergence.

```python
from tensorflow import keras

model = keras.Sequential([
    keras.layers.Conv2D(64, (3, 3), padding='same', input_shape=(32, 32, 3)),
    keras.layers.BatchNormalization(),
    keras.layers.Activation('relu'),
    keras.layers.Conv2D(64, (3, 3), padding='same'),
    keras.layers.BatchNormalization(),
    keras.layers.Activation('relu'),
    keras.layers.MaxPooling2D(2, 2),
    keras.layers.Flatten(),
    keras.layers.Dense(10, activation='softmax'),
])
```

---

## 8. Optimizers

### Adam — Default Starting Point

Adam adapts the learning rate for each parameter using estimates of first and second moments of gradients. Start with Adam at `lr=1e-3`. It converges fast and requires minimal tuning.

```python
from tensorflow import keras

optimizer = keras.optimizers.Adam(
    learning_rate=1e-3,
    beta_1=0.9,    # decay for first moment (gradient)
    beta_2=0.999,  # decay for second moment (squared gradient)
    epsilon=1e-7,
)

model.compile(optimizer=optimizer, loss='sparse_categorical_crossentropy',
              metrics=['accuracy'])
```

### SGD with Momentum

Vanilla SGD often generalizes better than Adam for image classification (final model quality). Momentum accumulates gradient history to smooth updates. Nesterov momentum looks ahead before updating.

```python
from tensorflow import keras

sgd = keras.optimizers.SGD(
    learning_rate=0.01,
    momentum=0.9,
    nesterov=True,
)

model.compile(optimizer=sgd, loss='categorical_crossentropy', metrics=['accuracy'])
```

### Learning Rate Schedules

Decaying the learning rate over time helps the model converge to a better minimum. Cosine decay is popular for training from scratch; exponential decay suits fine-tuning.

```python
from tensorflow import keras

# Cosine decay: lr decreases from initial_lr to 0 over decay_steps
lr_schedule = keras.optimizers.schedules.CosineDecay(
    initial_learning_rate=1e-3,
    decay_steps=1000,
)

# Exponential decay
lr_schedule_exp = keras.optimizers.schedules.ExponentialDecay(
    initial_learning_rate=1e-2,
    decay_steps=500,
    decay_rate=0.96,
    staircase=True,   # step-wise instead of continuous
)

optimizer = keras.optimizers.Adam(learning_rate=lr_schedule)
model.compile(optimizer=optimizer, loss='mse')
```

### RMSprop

Good default for RNNs and problems with non-stationary objectives. Maintains a moving average of squared gradients to normalize the gradient.

```python
from tensorflow import keras

rms = keras.optimizers.RMSprop(learning_rate=1e-3, rho=0.9)
model.compile(optimizer=rms, loss='binary_crossentropy', metrics=['accuracy'])
```

---

## 9. Saving and Loading

### Save and Load the Full Model

`model.save()` stores the architecture, weights, and optimizer state. This is the safest format — you can resume training and run inference from the same file.

```python
from tensorflow import keras

# Save — prefer .keras format (TF2.x native)
model.save('my_model.keras')

# Load — works even in a new Python session
loaded_model = keras.models.load_model('my_model.keras')

# Inference immediately available
predictions = loaded_model.predict(X_test)
```

### Save and Load Weights Only

Use when you want to save disk space or transfer weights into a model with the same architecture. You must rebuild the identical architecture before loading.

```python
from tensorflow import keras

# Save only the weights
model.save_weights('model_weights.weights.h5')

# Rebuild architecture, then load weights
new_model = build_my_model()   # must be identical architecture
new_model.load_weights('model_weights.weights.h5')
```

### SavedModel Format (TensorFlow Serving)

The SavedModel format exports the computation graph and is required for TF Serving and TF Lite conversion.

```python
import tensorflow as tf

# Export as SavedModel directory
model.export('saved_model_dir/')

# Load
loaded = tf.saved_model.load('saved_model_dir/')
infer = loaded.signatures['serving_default']
```

### Model Config — Architecture Only

Serialize just the architecture to JSON so you can reconstruct the model without the original Python class definition.

```python
from tensorflow import keras
import json

# Serialize architecture
config_json = model.to_json()

# Reconstruct architecture (no weights)
reconstructed = keras.models.model_from_json(config_json)

# Then load weights separately
reconstructed.load_weights('model_weights.weights.h5')
```

---

## 10. Custom Training Loop

### GradientTape

Use when `model.fit()` is not flexible enough: custom loss combinations, multi-model training (GANs), gradient clipping, or per-step logging. `GradientTape` records operations so TensorFlow can compute gradients automatically.

```python
import tensorflow as tf
from tensorflow import keras

model = keras.Sequential([
    keras.layers.Dense(64, activation='relu', input_shape=(20,)),
    keras.layers.Dense(1),
])

optimizer = keras.optimizers.Adam(1e-3)
loss_fn = keras.losses.MeanSquaredError()

@tf.function   # compile to graph for speed
def train_step(x_batch, y_batch):
    with tf.GradientTape() as tape:
        predictions = model(x_batch, training=True)
        loss = loss_fn(y_batch, predictions)

    gradients = tape.gradient(loss, model.trainable_variables)
    optimizer.apply_gradients(zip(gradients, model.trainable_variables))
    return loss

# Training loop
import numpy as np
X = np.random.randn(500, 20).astype('float32')
y = np.random.randn(500, 1).astype('float32')
dataset = tf.data.Dataset.from_tensor_slices((X, y)).batch(32)

for epoch in range(5):
    for x_batch, y_batch in dataset:
        loss = train_step(x_batch, y_batch)
    print(f'Epoch {epoch + 1}, Loss: {loss.numpy():.4f}')
```

### Custom Gradient Clipping

Prevents gradient explosion, which is common in RNNs and very deep networks.

```python
import tensorflow as tf
from tensorflow import keras

optimizer = keras.optimizers.Adam(1e-3)
loss_fn = keras.losses.SparseCategoricalCrossentropy()

def train_step(x_batch, y_batch):
    with tf.GradientTape() as tape:
        logits = model(x_batch, training=True)
        loss = loss_fn(y_batch, logits)

    gradients = tape.gradient(loss, model.trainable_variables)
    # Clip by global norm — prevents any single gradient from dominating
    gradients, _ = tf.clip_by_global_norm(gradients, clip_norm=1.0)
    optimizer.apply_gradients(zip(gradients, model.trainable_variables))
    return loss
```

---

## 11. Transfer Learning

### Load a Pretrained Base Model

Pretrained models (trained on ImageNet with millions of images) capture general visual features. `include_top=False` removes the original classifier so you can attach your own. `weights='imagenet'` downloads the pretrained weights.

```python
from tensorflow import keras

base_model = keras.applications.MobileNetV2(
    input_shape=(224, 224, 3),
    include_top=False,       # remove the original 1000-class classifier
    weights='imagenet',
)
```

### Freeze the Base and Train Only the Head

Freezing prevents the pretrained weights from being overwritten during initial training. Train only the new classification head first — this is faster and avoids destroying the pretrained features.

```python
from tensorflow import keras

base_model = keras.applications.MobileNetV2(
    input_shape=(224, 224, 3), include_top=False, weights='imagenet'
)
base_model.trainable = False   # freeze all layers in the base

inputs = keras.Input(shape=(224, 224, 3))
# MobileNetV2 expects inputs scaled to [-1, 1]
x = keras.applications.mobilenet_v2.preprocess_input(inputs)
x = base_model(x, training=False)   # training=False keeps BN in inference mode
x = keras.layers.GlobalAveragePooling2D()(x)
x = keras.layers.Dropout(0.3)(x)
outputs = keras.layers.Dense(5, activation='softmax')(x)   # 5 custom classes

model = keras.Model(inputs, outputs)

model.compile(optimizer=keras.optimizers.Adam(1e-3),
              loss='sparse_categorical_crossentropy',
              metrics=['accuracy'])
```

### Fine-Tuning — Unfreeze and Re-train at Low LR

After the head converges, unfreeze the top layers of the base and continue training at a very low learning rate. This adapts the high-level features to your specific domain without destroying the low-level features.

```python
from tensorflow import keras

# After initial head training converges...
base_model.trainable = True

# Freeze early layers, unfreeze from layer 100 onward
fine_tune_at = 100
for layer in base_model.layers[:fine_tune_at]:
    layer.trainable = False

# Re-compile with a much lower learning rate
model.compile(
    optimizer=keras.optimizers.Adam(1e-5),   # 100x lower than head training
    loss='sparse_categorical_crossentropy',
    metrics=['accuracy'],
)

model.fit(train_dataset, epochs=10, validation_data=val_dataset)
```

### Built-in Preprocessing Layers

Use Keras preprocessing layers so normalization is part of the model — no risk of train/test preprocessing mismatch.

```python
from tensorflow import keras

# Rescaling — maps [0, 255] to [0, 1]
rescale = keras.layers.Rescaling(scale=1.0 / 255)

# Data augmentation — only applied during training
augment = keras.Sequential([
    keras.layers.RandomFlip('horizontal'),
    keras.layers.RandomRotation(0.1),
    keras.layers.RandomZoom(0.1),
])

inputs = keras.Input(shape=(224, 224, 3))
x = rescale(inputs)
x = augment(x)
# ... rest of model
```

---

## 12. Data Loading with tf.data

### from_tensor_slices — In-Memory Data

Converts NumPy arrays or tensors into a dataset. The fundamental building block for tf.data pipelines.

```python
import tensorflow as tf
import numpy as np

X = np.random.randn(1000, 20).astype('float32')
y = np.random.randint(0, 2, size=(1000,)).astype('int32')

dataset = tf.data.Dataset.from_tensor_slices((X, y))
print(dataset.element_spec)
# Output: (TensorSpec(shape=(20,), dtype=tf.float32),
#          TensorSpec(shape=(), dtype=tf.int32))
```

### shuffle, batch, prefetch — The Standard Pipeline

This three-step pattern is the correct order for efficient training. Shuffle before batching to randomize across the full dataset; prefetch overlaps data loading with GPU computation.

```python
import tensorflow as tf

BATCH_SIZE = 32
BUFFER_SIZE = 1000   # how many samples to load into memory for shuffling

train_dataset = (
    tf.data.Dataset.from_tensor_slices((X_train, y_train))
    .shuffle(buffer_size=BUFFER_SIZE, seed=42)
    .batch(BATCH_SIZE)
    .prefetch(tf.data.AUTOTUNE)   # AUTOTUNE picks the optimal prefetch count
)

val_dataset = (
    tf.data.Dataset.from_tensor_slices((X_val, y_val))
    .batch(BATCH_SIZE)
    .prefetch(tf.data.AUTOTUNE)
    # No shuffle for validation
)

model.fit(train_dataset, validation_data=val_dataset, epochs=20)
```

### Loading Images from Disk

`image_dataset_from_directory` reads images organized into class-named subdirectories. It handles resizing, batching, and optional validation split.

```python
from tensorflow import keras

# Directory structure:
# data/train/cats/  ← images
# data/train/dogs/  ← images

train_ds = keras.utils.image_dataset_from_directory(
    'data/train/',
    image_size=(224, 224),
    batch_size=32,
    label_mode='int',    # integer labels; use 'categorical' for one-hot
    seed=42,
    validation_split=0.2,
    subset='training',
)

val_ds = keras.utils.image_dataset_from_directory(
    'data/train/',
    image_size=(224, 224),
    batch_size=32,
    label_mode='int',
    seed=42,
    validation_split=0.2,
    subset='validation',
)

# Add AUTOTUNE prefetch
train_ds = train_ds.prefetch(tf.data.AUTOTUNE)
val_ds = val_ds.prefetch(tf.data.AUTOTUNE)
```

### map — Apply Transformations

Use `.map()` to apply any preprocessing function element-wise. Run it before `.batch()` to work on individual samples, or after to work on full batches.

```python
import tensorflow as tf

def normalize(image, label):
    image = tf.cast(image, tf.float32) / 255.0
    return image, label

def augment(image, label):
    image = tf.image.random_flip_left_right(image)
    image = tf.image.random_brightness(image, max_delta=0.1)
    return image, label

train_ds = (
    tf.data.Dataset.from_tensor_slices((image_paths, labels))
    .map(load_image, num_parallel_calls=tf.data.AUTOTUNE)
    .map(normalize, num_parallel_calls=tf.data.AUTOTUNE)
    .map(augment, num_parallel_calls=tf.data.AUTOTUNE)
    .shuffle(500)
    .batch(32)
    .prefetch(tf.data.AUTOTUNE)
)
```

### cache — Speed Up Repeated Epochs

`.cache()` stores the dataset in memory (or on disk) after the first epoch. Without it, disk reads happen every epoch. Place it after expensive operations but before shuffling.

```python
import tensorflow as tf

train_ds = (
    tf.data.Dataset.from_tensor_slices((X_train, y_train))
    .map(preprocess, num_parallel_calls=tf.data.AUTOTUNE)
    .cache()                  # cache after preprocessing — only done once
    .shuffle(1000)
    .batch(32)
    .prefetch(tf.data.AUTOTUNE)
)
```

---

> [!tip]
> **Quick decision guide:**
> - Structured/tabular data → Sequential with Dense layers, Adam optimizer, start simple
> - Image classification → CNN or pretrained MobileNetV2/EfficientNet with transfer learning
> - Sequence/time series → LSTM or 1D Conv; normalize inputs to zero mean, unit variance
> - NLP → Embedding layer + LSTM, or use a pretrained HuggingFace transformer
> - Any training plateaus → check learning rate first, then regularization, then architecture

> [!warning]
> - Always call `model.compile()` after changing `layer.trainable` during fine-tuning — Keras needs to recompute the gradient graph.
> - `validation_split` takes the last fraction of your data in order. For time series this leaks future data. Use `validation_data` with an explicit split instead.
> - `model.predict()` returns NumPy arrays; `model(x, training=False)` returns a tensor. For inference pipelines, prefer `predict()`.
> - The `.keras` format is the recommended save format in TF2.x. The older `.h5` format has limitations with custom objects.
