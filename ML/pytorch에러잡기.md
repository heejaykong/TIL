models.py
```
from torchvision import models as models
import torch.nn as nn

def model(pretrained, requires_grad):
    # model = models.resnet50(progress=True, pretrained=pretrained)
    model = models.resnet18(progress=True, pretrained=pretrained)
    # to freeze the hidden layers
    if requires_grad == False:
        for param in model.parameters():
            param.requires_grad = False
    # to train the hidden layers
    elif requires_grad == True:
        for param in model.parameters():
            param.requires_grad = True
    # make the classification layer learnable
    # we have 13 classes in total
    model.fc = nn.Linear(2048, 13)  <<< 이 부분을 고쳐줘야 했다. resnet18에 맞는 in_features값인 512로.
    return model
```

train.py 일부 epoch 코드
```
# start the training and validation
train_loss = []
valid_loss = []

for epoch in range(epochs):  #epochs
    print(f"Epoch {epoch+1} of {epochs}")
    train_epoch_loss = train(
        model, train_loader, optimizer, scheduler, criterion, train_data, device
    )
    print("train_epoch_loss", train_epoch_loss)
    valid_epoch_loss = validate(
        model, valid_loader, criterion, valid_data, device
    )
    print("valid_epoch_loss", valid_epoch_loss)
    train_loss.append(train_epoch_loss)
    valid_loss.append(valid_epoch_loss)
    print(f"Train Loss: {train_epoch_loss:.4f}")
    print(f'Val Loss: {valid_epoch_loss:.4f}')
```

error message
```
Epoch 1 of 10
Training
  0%|          | 0/1875 [00:00<?, ?it/s]/usr/local/lib/python3.7/dist-packages/ipykernel_launcher.py:69: UserWarning: To copy construct from a tensor, it is recommended to use sourceTensor.clone().detach() or sourceTensor.clone().detach().requires_grad_(True), rather than torch.tensor(sourceTensor).
/usr/local/lib/python3.7/dist-packages/torch/nn/functional.py:718: UserWarning: Named tensors and all their associated APIs are an experimental feature and subject to change. Please do not use them for anything important until they are released as stable. (Triggered internally at  /pytorch/c10/core/TensorImpl.h:1156.)
  return torch.max_pool2d(input, kernel_size, stride, padding, dilation, ceil_mode)
  0%|          | 0/1875 [00:17<?, ?it/s]
---------------------------------------------------------------------------
RuntimeError                              Traceback (most recent call last)
<ipython-input-16-3abb407fd403> in <module>()
      6     print(f"Epoch {epoch+1} of {epochs}")
      7     train_epoch_loss = train(
----> 8         model, train_loader, optimizer, scheduler, criterion, train_data, device
      9     )
     10     print("train_epoch_loss", train_epoch_loss)

6 frames
/usr/local/lib/python3.7/dist-packages/torch/nn/functional.py in linear(input, weight, bias)
   1845     if has_torch_function_variadic(input, weight):
   1846         return handle_torch_function(linear, (input, weight), input, weight, bias=bias)
-> 1847     return torch._C._nn.linear(input, weight, bias)
   1848 
   1849 

RuntimeError: CUDA error: CUBLAS_STATUS_INVALID_VALUE when calling `cublasSgemm( handle, opa, opb, m, n, k, &alpha, a, lda, b, ldb, &beta, c, ldc)`
```


### 참고
* [RuntimeError: CUDA error: CUBLAS_STATUS_INVALID_VALUE when calling `cublasSgemm( handle, opa, opb, m, n, k, &alpha, a, lda, b, ldb, &beta, c, ldc)`](https://discuss.pytorch.org/t/runtimeerror-cuda-error-cublas-status-invalid-value-when-calling-cublassgemm-handle-opa-opb-m-n-k-alpha-a-lda-b-ldb-beta-c-ldc/124544/6)
* [FINETUNING TORCHVISION MODELS - Resnet](https://pytorch.org/tutorials/beginner/finetuning_torchvision_models_tutorial.html#resnet)
