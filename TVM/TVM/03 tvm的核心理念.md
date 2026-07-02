既然TVM最后生成的CUDA kernel可以放到cuDNN上使用，就成了一个标准的kernel了,那么对于Nvidia来说这不是一件很好的事吗？只需要使用TVM来生成CUDA kernel不就行了吗？还需要招人来手动写kernel吗，这样效率不就很低下了吗？

**确实有道理，但是并不会，TVM生成的kernel也不会放到cuDNN库里，因为TVM 自动生成 kernel 的性能很高，但未必能超过人工精调的 cuDNN kernel。**



**更多的应用场景是：**使用TVM自动生成初版kernel，然后自行人工针对具体地硬件架构进行优化，最终形成性能极致、可靠的kernel会加入cuDNN库。



所以TVM的核心理念是提出一个初版然后由人工优化