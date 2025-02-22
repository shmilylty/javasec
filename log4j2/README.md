**注：**
```
# 本文仅供安全研究使用 👨‍✈️
这里想提一嘴，看到很多人在"疯狂"地测dnslog，其实还是希望在没有授权情况下最好不要进行漏洞探测/攻击行为，毕竟"自律是自由的基础"!  👨‍✈️ 
```

### 漏洞分析

根据流传的payload搭建测试环境

- log4j_rce.java

```java
import org.apache.logging.log4j.LogManager;
import org.apache.logging.log4j.Logger;

public class log4j_rce {
    private static final Logger logger = LogManager.getLogger(log4j_rce.class);
    public static void main(String[] args) {
        logger.error("暂时打码处理"}");
    }
}
```

一步一步跟进，最后跟到
- org.apache.logging.log4j.core.lookup.Interpolator#lookup

![image](vulnerability-research.assets/145439561-9dc12330-6705-4297-aeef-42413a4edc21.png)

似乎就是这里，测下断点，调试

![image](vulnerability-research.assets/145439835-a4d042d7-80b9-47ca-a65b-8dcb24ee6bd1.png)

答案呼之欲出，lookup + jndi！

其实这里可以触发的不只是error,默认情况下fatal也可以，即便实际的业务场景肯定会有所不同(只会更多)。

### 漏洞复现

起一个恶意的LDAPRefServer、恶意类Evil

![image](vulnerability-research.assets/145442280-7bfa4038-276e-414b-b0b6-2c1dc6e92e74.png)

然后触发即可

![image](vulnerability-research.assets/145458787-4acd92c0-50db-40dc-b4cb-6d49014e53ac.png)
