---
title: MLP的pytorch实现
tags:
categories:
  - 人工智能
toc: true
mathjax: true
top: false
comments: true
copyright: true
date: 2022-05-16 21:40:45
---

# 1 问题

xxx

# 2 实现

```python
from sklearn.datasets import load_digits
from sklearn.model_selection import train_test_split
import numpy as np
import torch
import torch.nn as nn
import matplotlib.pyplot as plt
from torch.utils.data import Dataset, DataLoader
from sklearn.metrics import accuracy_score, precision_score, recall_score, f1_score, classification_report, confusion_matrix
from sklearn.preprocessing import label_binarize
from sklearn.metrics import roc_curve, auc
import time

np.random.seed(0)
torch.manual_seed(0)
torch.cuda.manual_seed_all(0)

print("加载数据")
digits = load_digits()
data, label = digits.data, digits.target
# print(data.shape, label.shape)
train_data, test_data, train_label, test_label = train_test_split(data, label, test_size=.3, random_state=123)
print('训练数据：', train_data.shape)
print('测试数据：', test_data.shape)

print("定义相关参数")
epochs = 30
batch_size = train_data.shape[0]
device = torch.device("cuda:0" if torch.cuda.is_available() else "cpu")
input_dim = data.shape[1]
hidden_dim = 256
output_dim = len(set(label))

print("构建数据集")
class DigitsDataset(Dataset):
  def __init__(self, input_data, input_label):
    data = []
    for i,j in zip(input_data, input_label):
      data.append((i,j))
    self.data = data

  def __len__(self):
    return len(self.data)

  def __getitem__(self, index):
    d, l = self.data[index]
    return d, l

trainDataset = DigitsDataset(train_data, train_label)
testDataset = DigitsDataset(test_data, test_label)
# print(trainDataset[0])
# print(trainDataset[0])
trainDataLoader = DataLoader(trainDataset, batch_size=batch_size, shuffle=True, num_workers=0)  # 2改为0
testDataLoader = DataLoader(testDataset, batch_size=batch_size, shuffle=False, num_workers=0)

class Model(nn.Module):
  def __init__(self, input_dim, hidden_dim, output_dim):
    super(Model, self).__init__()
    self.fc1 = nn.Linear(input_dim, hidden_dim)
    self.relu = nn.ReLU()
    self.fc2 = nn.Linear(hidden_dim, output_dim)

  def forward(self, x):
    x = self.fc1(x)
    x = self.relu(x)
    x = self.fc2(x)
    return x

model = Model(input_dim, hidden_dim, output_dim)
print(model)
model.to(device)

print("定义损失函数、优化器")
criterion = nn.CrossEntropyLoss()
optimizer = torch.optim.SGD(model.parameters(), lr=0.01, momentum=0.9, weight_decay=1e-4)

print("初始化相关参数")
for param in model.parameters():
  nn.init.normal_(param, mean=0, std=0.01)


print("开始训练主循环")
total_step = len(trainDataLoader)

model.train()
for epoch in range(epochs):
  tot_loss = 0.0
  tot_acc = 0.0
  train_preds = []
  train_trues = []
  # model.train()
  for i,(train_data_batch, train_label_batch) in enumerate(trainDataLoader):
    train_data_batch = train_data_batch.float().to(device) # 将double数据转换为float
    train_label_batch = train_label_batch.to(device)
    outputs = model(train_data_batch)
    # _, preds = torch.max(outputs.data, 1)
    loss = criterion(outputs, train_label_batch.type(torch.LongTensor))
    # print(loss)
    #反向传播优化网络参数
    loss.backward()
    optimizer.step()
    optimizer.zero_grad()
    #累加每个step的损失
    tot_loss += loss.data
    train_outputs = outputs.argmax(dim=1)

    train_preds.extend(train_outputs.detach().cpu().numpy())
    train_trues.extend(train_label_batch.detach().cpu().numpy())

    # tot_acc += (outputs.argmax(dim=1) == train_label_batch).sum().item()

  sklearn_accuracy = accuracy_score(train_trues, train_preds)
  sklearn_precision = precision_score(train_trues, train_preds, average='micro')
  sklearn_recall = recall_score(train_trues, train_preds, average='micro')
  sklearn_f1 = f1_score(train_trues, train_preds, average='micro')
  print("[sklearn_metrics] Epoch:{} loss:{:.4f} accuracy:{:.4f} precision:{:.4f} recall:{:.4f} f1:{:.4f}".format(epoch, tot_loss, sklearn_accuracy, sklearn_precision, sklearn_recall, sklearn_f1))


def get_confusion_matrix(trues, preds):
  labels = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
  conf_matrix = confusion_matrix(trues, preds, labels=labels)
  return conf_matrix


def plot_confusion_matrix(conf_matrix):
  plt.imshow(conf_matrix, cmap=plt.cm.Greens)
  indices = range(conf_matrix.shape[0])
  labels = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
  plt.xticks(indices, labels)
  plt.yticks(indices, labels)
  plt.colorbar()
  plt.xlabel('y_pred')
  plt.ylabel('y_true')
  # 显示数据
  for first_index in range(conf_matrix.shape[0]):
    for second_index in range(conf_matrix.shape[1]):
      plt.text(first_index, second_index, conf_matrix[first_index, second_index])
  plt.savefig('heatmap_confusion_matrix.jpg')
  plt.show()


test_preds = []
test_trues = []
model.eval()
with torch.no_grad():
  for i,(test_data_batch, test_data_label) in enumerate(testDataLoader):
    test_data_batch = test_data_batch.float().to(device) # 将double数据转换为float
    test_data_label = test_data_label.to(device)
    test_outputs = model(test_data_batch)
    test_outputs = test_outputs.argmax(dim=1)
    test_preds.extend(test_outputs.detach().cpu().numpy())
    test_trues.extend(test_data_label.detach().cpu().numpy())

  sklearn_precision = precision_score(test_trues, test_preds, average='micro')
  sklearn_recall = recall_score(test_trues, test_preds, average='micro')
  sklearn_f1 = f1_score(test_trues, test_preds, average='micro')
  print(classification_report(test_trues, test_preds))
  conf_matrix = get_confusion_matrix(test_trues, test_preds)
  print(conf_matrix)
  plot_confusion_matrix(conf_matrix)
  print("[sklearn_metrics] accuracy:{:.4f} precision:{:.4f} recall:{:.4f} f1:{:.4f}".format(sklearn_accuracy, sklearn_precision, sklearn_recall, sklearn_f1))


# Press the green button in the gutter to run the script.
if __name__ == '__main__':
    start = time.time()
    print('train-mlp')
    end = time.time()
    print((end - start) / 60, "min")  # 秒
```

