---
sort: 4
---

# PSM相关内容

**平台特定模型（PSM）：UDP/IP**
## 1. 简介
- 平台特定模型（PSM）将协议平台独立模型（PIM）映射到UDP/IP。UDP/IP作为DDS应用的传输协议具有多种优势，包括通用可用性、轻量级、提供尽力而为服务、无连接、行为可预测以及支持可扩展性和组播等。

## 2. 符号约定
- **名称空间**：文档中的定义属于“RTPS”名称空间，但为方便阅读，定义和类中省略了名称空间前缀。
- **IDL表示和CDR线表示**：常使用IDL定义结构，在网络传输时使用相应的CDR表示。
- **位和字节表示**：使用特定的记号表示字节和字节流，明确了最高有效位（MSB）和最低有效位（LSB）以及字节流的顺序。

## 3. RTPS类型映射
- **全局唯一标识符（GUID）**
     - **GuidPrefix_t映射**：PSM将GuidPrefix_t映射为12字节的结构，定义了未知常量的映射值。
     - **EntityId_t映射**：将EntityId_t映射为特定结构，包含实体键和实体类型字段，定义了不同实体类型的值以及未知常量的映射值，同时说明了实体类型编码方式和实体ID的选择规则。
     - **预定义实体ID**：介绍了RTPS协议中预定义的实体ID及其PSM映射值，包括参与者、各种内置主题写入器和读取器等的实体ID，还提到其他规范可能保留的实体ID。
     - **弃用的实体ID**：列出了协议2.2版本中弃用的实体ID以及使用注意事项。
     - **GUID_t映射**：将GUID_t映射为包含GuidPrefix_t和EntityId_t的结构，说明了如何确保guidPrefix在DDS域内的唯一性以及未知常量的映射值。
- **消息内或内置主题数据中出现的类型**
     - 定义了多种类型的PSM映射，如Time_t、Duration_t、VendorId_t、SequenceNumber_t、Locator_t、ReliabiliyKind_t、Count_t、ProtocolVersion_t、KeyHash_t、StatusInfo_t、ParameterId_t、ContentFilterProperty_t、FilterResult_t、FilterSignature_t、Property_t、OriginalWriterInfo_t、GroupDigest_t等，包括其结构定义和相关常量的映射。

## 4. RTPS消息映射
 - **整体结构**：消息由头部和多个子消息组成，PSM将子消息在消息中按32位边界对齐，消息长度由底层传输确定（如UDP/IP中为UDP有效载荷长度）。
 - **PIM子消息元素的映射**
     - 详细介绍了各种子消息元素的PSM映射，包括EntityId、GuidPrefix、VendorId、ProtocolVersion、SequenceNumber、SequenceNumberSet、FragmentNumber、FragmentNumberSet、Timestamp、LocatorList、ParameterList、SerializedPayload、Count、GroupDigest等，给出了IDL定义和线表示形式。
 - **额外的子消息元素**：介绍了UDP PSM引入的额外子消息元素，如LocatorUDPv4，包括其结构、映射和线表示。
 - **RTPS头部的映射**：展示了RTPS头部的PSM映射结构，包括协议版本、供应商ID和前缀，强调了头部结构在协议主版本2中不能改变，且供应商需使用保留的供应商ID。
 - **RTPS子消息的映射**
     - **子消息头部**：将子消息头部映射为包含子消息ID、标志和长度的结构，介绍了子消息ID的取值范围（协议特定和供应商特定）以及标志位（如字节序标志EndiannessFlag等）和长度字段的含义和表示方式。
     - **各种子消息的映射**：详细介绍了AckNack、Data、DataFrag、Gap、HeartBeat、HeartBeatFrag、InfoDestination、InfoReply、InfoSource、InfoTimestamp、Pad、NackFrag、InfoReplyIp4等子消息的逻辑内容及其PSM映射的线表示形式，包括各子消息引入的标志位及其含义和映射方式。

## 5. 映射到UDP/IP传输消息
当RTPS在UDP/IP上使用时，一个消息是一个UDP/IP数据报的有效载荷。

## 6. RTPS协议映射
 - **默认定位器**
     - **发现流量**：介绍了参与者和端点发现协议产生的发现流量所使用的端口映射，包括SPDP和SEDP内置端点的端口映射，以及如何通过参数避免端口冲突，还给出了默认的端口参数值。
     - **用户流量**：说明了用户定义端点的流量所使用的端口表达式，以及端点可以选择不使用默认端口的情况。
     - **默认端口号**：介绍了端口号表达式中使用的参数，并给出了默认参数值以及这些默认值所能支持的域和参与者数量。
     - **简单参与者发现协议的默认设置**：包括默认的多播地址和公告速率等设置。
 - **内置端点的数据表示**
     - **参与者消息数据内置端点的数据表示**：将ParticipantMessageData类型映射为特定的IDL结构，介绍了类型中kind字段的保留值以及数据字段的长度要求，并给出了线表示形式。
     - **简单发现协议内置端点的数据表示**：将SPDPdiscoveredParticipantData、DiscoveredWriterData、DiscoveredReaderData和DiscoveredTopicData等类型映射为相应的IDL结构，说明了如何使用ParameterList子消息元素表示序列化数据以实现QoS可扩展性和版本间的互操作性，还介绍了参数ID空间的划分、保留情况以及不同内置端点应省略的参数，同时给出了常见参数ID的值、映射和默认值。
 - **参数ID定义**
     - 介绍了用于表示内联QoS的参数ID定义，包括Content filter info、Coherent set、Group Coherent Set、Group Sequence Number、Publisher Writer Info、Secure Publisher Writer Info、Original Writer Info、KeyHash、StatusInfo_t等参数的含义、线表示形式以及相关的计算和编码规则。
 - **被协议弃用的参数ID**：列出了被协议弃用的参数ID以及弃用的版本，提醒在使用相应协议版本时应避免使用这些参数，除非与旧版本保持相同含义。
