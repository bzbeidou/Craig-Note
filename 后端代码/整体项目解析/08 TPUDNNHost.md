### 1、TPUDNNHost 

```c++
class TPUDNNHost : public TPUDNNInterface
{
private:
    using Runtime = RuntimePolicy<DeviceConfig>;
    using TPUDNNDevice = TPUDevice<
        RuntimePolicy,
        DeviceConfig,
        OSPolicy>;
    using PMU = PMUPolicy<TPUDNNDevice, DeviceConfig, Runtime>;

    template <typename T, typename = void>
    struct isProfileSupported : std::false_type {};
    template <typename T>
    struct isProfileSupported<T, std::void_t<typename T::gdma_pmu_item_t>> : std::true_type {};

    template <typename T, typename = void>
    struct isCacheSupported : std::false_type {};
    template <typename T>
    struct isCacheSupported<T, std::void_t<decltype(T::CacheEngineNum)>> : std::true_type {};

    using Profile = ProfilePolicy<TPUDNNDevice, DeviceConfig>;
    using Memcpy = MemcpyPolicy<Runtime>;
    using Cache = CachePolicy<DeviceConfig, TPUDNNDevice, typename TPUDNNDevice::Stream, Profile>;

    TPUDNNDevice mDevice;
    typename TPUDNNDevice::Stream mStream;
    Cache mCache;
    std::shared_ptr<Profile> mProfile;
    bool mCacheEnabled;
    typedef struct {
        uint64_t addr;
        uint64_t size;
        uint64_t info;
    } mem_guard_param_t;
```

### 2、TPUDevice

```c++
template <template <class> class RuntimePolicy, class DevicePolicy, class OSPolicy>
class TPUDevice : public RuntimePolicy<DevicePolicy>, public OSPolicy
{
private:
    void *mStartingAddr;
    using Runtime = RuntimePolicy<DevicePolicy>;

public:
    using Stream = TPUStream<DevicePolicy, typename Runtime::Launcher>;
    using DeviceConfig = DevicePolicy;

    TPUDevice(int deviceID) : Runtime(deviceID)
    {
        mStartingAddr = OSPolicy::reserveAddr(DevicePolicy::AddrSpaceSize);
    }
```

