<template>
    <div class="radius card" :style="{ borderRadius: `var(--el-border-radius-round)` }">
        <div style="text-align: center;">
            <transition name="el-fade-in">
                <div v-if="ipInfo.local && ipInfo.local.country && ipInfo.local.country.code == 'CN'">
                    <el-tooltip class="item" effect="dark" :content="ipInfo.local.ip" placement="top">
                        <div @click.stop="onQuery(ipInfo.local.ip)">
                            <el-tag style="width: 50px;" class="ml-2" type="success">{{
                                ipInfo.layLocal?ipInfo.layLocal+"ms":"-ms" }}</el-tag>
                            <el-text style="cursor: pointer;margin-left: 5px;white-space:nowrap;vertical-align: -1px;"
                                class="font-background">{{ ipInfo.local.show.join(" ") }}</el-text>
                        </div>
                    </el-tooltip>
                </div>
            </transition>
            <transition name="el-fade-in">
                <div v-if="ipInfo.cloudflare && ipInfo.cloudflare.country && ipInfo.cloudflare.country.code != 'CN'">
                    <el-tooltip class="item" effect="dark" :content="ipInfo.cloudflare.ip" placement="top">
                        <div @click.stop="onQuery(ipInfo.cloudflare.ip)">
                            <el-tag style="width: 50px;" class="ml-2" type="success">{{
                                ipInfo.layCloudflare?ipInfo.layCloudflare+"ms":"-ms" }}</el-tag>
                            <el-text style="cursor: pointer;margin-left: 5px;white-space:nowrap;vertical-align: -1px;"
                                class="font-background">{{ ipInfo.cloudflare.show.join(" ") }}</el-text>
                        </div>
                    </el-tooltip>
                </div>
            </transition>
            <transition name="el-fade-in">
                <div v-if="!ipInfo.local && !ipInfo.cloudflare" v-loading="true">
                    <el-tooltip class="item" effect="dark" content="" placement="top">
                        <div>
                            <el-text style="cursor: pointer;margin-left: 5px;white-space:nowrap;vertical-align: -1px;"
                                class="font-background">正在加载...</el-text>
                        </div>
                    </el-tooltip>
                </div>
            </transition> 
        </div>
    </div>
    <el-dialog align-center style="width: 95vw;max-width: 600px;max-height: 85vh;" v-model="queryWindow" title="IP查询">
        <el-input v-model="ipInput" style="max-width: 600px" placeholder="请输入IPV4/IPV6地址" clearable autocomplete="new-password">
            <template #append><el-button type="primary" :icon="Search" circle  @click="onQuery(ipInput)"/></template>
        </el-input>
         <table class="ip-table" v-loading="isQuerying">
            <tbody>
                <tr>
                    <td @click="copy(ipRet.ip || '')">{{ ipRet.ip }}</td>
                </tr>
                <tr v-if="ipRet.country">
                    <td>
                        {{ ipRet.country?.name }} ({{ ipRet.country?.code }})
                    </td>
                </tr>
            </tbody>
         </table>
    </el-dialog>
</template>

<script lang="ts" setup>
const props = defineProps({
    isVisible: Boolean
})
import { reactive,ref, type Ref } from 'vue'
import { ElMessage } from 'element-plus'
import { toClipboard } from '@soerenmartius/vue3-clipboard'
import { Search } from '@element-plus/icons-vue'
const queryWindow = ref(false)
const isQuerying = ref(false)
const ipInput = ref("")
const ipRet: Ref<{ 
  ip?: string; 
  country?: { name?: string; code?: string }; 
  show?: string[]; 
}> = ref({});
const failureRef = ref(false); // 将 failure 变量改为响应式
const onQuery = async(ip:string) => {
    queryWindow.value = true;
    try {
        const ipv4 = /^((25[0-5]|2[0-4]\d|[01]?\d\d?)\.){3}(25[0-5]|2[0-4]\d|[01]?\d\d?)$/;
        const ipv6 = /^([\da-fA-F]{1,4}:){6}((25[0-5]|2[0-4]\d|[01]?\d\d?)\.){3}(25[0-5]|2[0-4]\d|[01]?\d\d?)$|^::([\da-fA-F]{1,4}:){0,4}((25[0-5]|2[0-4]\d|[01]?\d\d?)\.){3}(25[0-5]|2[0-4]\d|[01]?\d\d?)$|^([\da-fA-F]{1,4}:):([\da-fA-F]{1,4}:){0,3}((25[0-5]|2[0-4]\d|[01]?\d\d?)\.){3}(25[0-5]|2[0-4]\d|[01]?\d\d?)$|^([\da-fA-F]{1,4}:){2}:([\da-fA-F]{1,4}:){0,2}((25[0-5]|2[0-4]\d|[01]?\d\d?)\.){3}(25[0-5]|2[0-4]\d|[01]?\d\d?)$|^([\da-fA-F]{1,4}:){3}:([\da-fA-F]{1,4}:){0,1}((25[0-5]|2[0-4]\d|[01]?\d\d?)\.){3}(25[0-5]|2[0-4]\d|[01]?\d\d?)$|^([\da-fA-F]{1,4}:){4}:((25[0-5]|2[0-4]\d|[01]?\d\d?)\.){3}(25[0-5]|2[0-4]\d|[01]?\d\d?)$|^([\da-fA-F]{1,4}:){7}[\da-fA-F]{1,4}$|^:((:[\da-fA-F]{1,4}){1,6}|:)$|^[\da-fA-F]{1,4}:((:[\da-fA-F]{1,4}){1,5}|:)$|^([\da-fA-F]{1,4}:){2}((:[\da-fA-F]{1,4}){1,4}|:)$|^([\da-fA-F]{1,4}:){3}((:[\da-fA-F]{1,4}){1,3}|:)$|^([\da-fA-F]{1,4}:){4}((:[\da-fA-F]{1,4}){1,2}|:)$|^([\da-fA-F]{1,4}:){5}:([\da-fA-F]{1,4})?$|^([\da-fA-F]{1,4}:){6}:$/;

        if (!ipv4.test(ip) && !ipv6.test(ip)) {
            throw new Error("请输入正确的IP地址");
        }
        isQuerying.value = true;
        // 直接调用 queryIp 而不是 cachedQuery，因为我们是在手动查询，可能需要最新数据
        // ipRet.value = await cachedQuery(ip); 
        ipRet.value = await queryIp(ip);
        isQuerying.value = false;
    } catch (error) {
        isQuerying.value = false; // 确保在错误时也关闭 loading 状态
        console.error("onQuery failed:", error);
        ElMessage.error(String(error));
    }
}
const ipInfo: {local:any, cloudflare:any,layLocal:any,layCloudflare:any} = reactive({local:null, cloudflare:null,layLocal:null,layCloudflare:null})
const copy = (ip: string) => {
    toClipboard(ip)
    ElMessage.success({
        dangerouslyUseHTMLString: true,
        message: `已经复制IP地址：<br><strong>${ip}</strong>`,
    })
}

// 使用 ipapi.co 免费 IP 查询服务
// 注意：ipapi.co 对于浏览器直接请求可能有 CORS 限制
// 尝试使用 JSONP 或代理方式，这里先用 no-cors 模式看是否能获取数据（但会是 opaque response）
async function queryIp(ip: string) {
    try {
        // 为了兼容性，我们使用一个支持 CORS 的免费 API
        // seeip.org 的正确 IP 地理位置查询端点是 /geoip/{IP}
        // 例如：https://api.seeip.org/geoip/8.8.8.8
        // // console.log("Trying alternative CORS-enabled API for IP:", ip);
        const fallbackRsp = await fetch(`https://api.seeip.org/geoip/${ip}`, {
            method: "GET",
            mode: "cors", // 明确使用 cors 模式
            redirect: "follow",
            referrerPolicy: "no-referrer"
        });
        
        if (!fallbackRsp.ok) {
            throw new Error(`seeip.org API request failed with status ${fallbackRsp.status}`);
        }
        
        let resp = await fallbackRsp.json();
        
        // 适配 seeip.org /geoip 端点返回的数据结构
        // 简化数据，只保留必要的字段
        const adaptedData = {
            ip: resp.ip || ip,
            country: {
                name: resp.country_name || '',
                code: resp.country_code || ''
            },
            // 添加 show 字段，只显示 IP 和国家
            show: [] 
        };
        
        // 填充 show 字段
        if (adaptedData.ip) {
            adaptedData.show.push(adaptedData.ip);
        }
        // 确保 country 对象存在且 name 非空
        if (adaptedData.country && adaptedData.country.name) {
            adaptedData.show.push(adaptedData.country.name);
        }
        // 如果 country.name 为空，但 code 存在，也可以显示 code
        else if (adaptedData.country && adaptedData.country.code) {
             adaptedData.show.push(`(${adaptedData.country.code})`);
        }
        
        return adaptedData;
        
    } catch (error) {
        console.error("IP query failed for:", ip, error);
        // 如果所有方法都失败，返回一个包含错误信息的对象
        return {
            ip: ip,
            show: [`Error: ${error.message}`],
            country: { code: '', name: '' }
        };
    }
}

const failure = ref(false); // 将 failure 变量改为响应式
async function cachedQuery(ip: string) {
    let ret = JSON.parse(localStorage.getItem("cache_ip_"+ip) || "{}")
    // 增加一个简单的有效性检查，确保缓存的数据结构大致正确
    if (!ret.ip || new Date().getTime() / 1000 - ret.time > 60 * 60 * 24){
        try {
            // 使用响应式的 failure 状态
            if (failureRef.value) { 
                 throw new Error("Previous query failed, skipping...");
            }
            ret = await queryIp(ip)
        } catch (error) {
            console.error("cachedQuery failed for IP:", ip, error);
            failureRef.value = true; // 设置响应式的 failure 状态
            // 抛出一个更具体的错误信息
            throw new Error("查询IP信息失败: " + (error.message || error)); 
        }
        ret['time'] = new Date().getTime() / 1000
        localStorage.setItem("cache_ip_"+ip, JSON.stringify(ret))
    }
    return ret
}

async function handleIP(ip: string) {
    try {
        // 直接调用 queryIp 获取最新数据
        const info = await queryIp(ip);
        // 确保 info 是一个对象
        if (typeof info !== 'object' || info === null) {
            console.error("handleIP: queryIp did not return a valid object for IP:", ip, info);
            return { show: [`Error: Invalid data for ${ip}`], ip: ip, country: { name: '', code: '' } };
        }
        
        // 确保 country 对象存在
                if (!info.country) {
                    info.country = { name: '', code: '' };
                }
                // 如果 queryIp 已经正确设置了 show 字段（只包含 IP 和国家），则直接返回
                // 不再需要额外的处理
                
                return info;
    } catch (error) {
        console.error("handleIP failed for IP:", ip, error.message);
        // 返回一个包含错误信息的对象
        return { show: [`Error: ${error.message}`], ip: ip }; 
    }
}

(async function watchLocalIp() {
    if (props.isVisible) {
        try {
            // 1. 首先获取本地公网 IP
            // 使用 api.ipify.org，它是一个广泛支持 CORS 的免费 IP 查询服务
            // 例如：https://api.ipify.org?format=json
            const response = await fetch('https://api.ipify.org?format=json', { 
                method: 'GET',
                mode: 'cors', // 明确指定 CORS 模式
                referrerPolicy: 'no-referrer' 
            });
            
            if (!response.ok) {
                throw new Error(`Failed to fetch local IP from ipify.org: ${response.status} ${response.statusText}`);
            }
            
            let resp = await response.json();
            // ipify.org ?format=json 返回 { "ip": "xxx.xxx.xxx.xxx" }
            const localIp = resp.ip;

            // 2. 使用获取到的 IP 调用我们新的 queryIp 函数 (即 seeip.org geoip) 查询详细信息
            if (localIp) {
                // 注意：这里直接调用 queryIp，避免缓存可能带来的问题
                const localIpInfo = await queryIp(localIp);
                // 只有在成功获取信息后才更新 reactive 对象
                if (localIpInfo && localIpInfo.ip) {
                    ipInfo.local = localIpInfo;
                }
            } else {
                 // console.warn("Could not extract local IP from ipify.org response.", resp);
            }

        } catch (error) {
            // // console.log("Local IP info fetch failed:", error);
        }
    }
    // 缩短间隔到5秒，避免过于频繁的请求
    setTimeout(watchLocalIp, 5000);
})();

const watchCloudflare = async(host: string) => {
    if (props.isVisible) {
        try {
            // 1. 首先通过 Cloudflare trace 获取 IP
            var start_timestamp = new Date().getTime();
            const response = await fetch(`https://${host}/cdn-cgi/trace`, { referrerPolicy: 'no-referrer' });
            const lay = new Date().getTime() - start_timestamp;
            let resp = await response.text();
            let ip_match = resp.match(/ip=([0-9a-f.:]+)/);
            if (ip_match && ip_match[1]) {
                const cfIp = ip_match[1];
                // 2. 使用获取到的 IP 调用我们新的 queryIp 函数 (即 seeip.org)
                // 注意：这里直接调用 queryIp，避免缓存可能带来的问题
                const cfIpInfo = await queryIp(cfIp); 
                // 只有在成功获取信息后才更新 reactive 对象
                if (cfIpInfo && cfIpInfo.ip) {
                     ipInfo.cloudflare = cfIpInfo; 
                     ipInfo.layCloudflare = lay;
                }
            } else {
                // console.warn("Could not extract Cloudflare IP from trace response.");
            }
        } catch (error) {
            // console.log("Cloudflare IP info fetch failed:", error);
        }
    }
    // 缩短间隔到5秒，避免过于频繁的请求
    setTimeout(watchCloudflare, 5000, host);
}

watchCloudflare("cp.cloudflare.com")
// watchCloudflare("chat.openai.com")

;async function getCNLay() {
    if (props.isVisible) {
        try {
            var start_timestamp = new Date().getTime();
            // 使用一个更通用且可能更快速的国内网站来测试延迟
            await fetch("https://www.baidu.com", { 
                method: "HEAD", 
                cache: "no-store", 
                mode: 'no-cors', 
                referrerPolicy: 'no-referrer' 
            });
            ipInfo.layLocal = new Date().getTime() - start_timestamp;
        } catch (error) {
            console.error("getCNLay failed:", error);
            ipInfo.layLocal = 0; // 确保在错误时也设置一个值
        }
    }
    setTimeout(getCNLay, 1000); // 缩短间隔到1秒，与 watchCloudflare 一致
}
</script>

<style scoped>
.font-background {
    color: #344357;
    font-size: 14px;
}

.card {
    max-width: 800px;
    height: fit-content;
    display: block;
    margin: 0 auto;
    background-color: #ffffff;
    padding: 2%
}

.ip-table {
    height: 100%;
    margin: 20px auto;
    padding: 10px;
    border: #ffffff 1px solid;
    border-radius: 10px;

    text-align: center;
}

@media (prefers-color-scheme: dark) {
    .card {
        background-color: rgb(18, 18, 18);
    }

    .font-background {
        color: rgb(193, 206, 230);
    }
}
</style>
