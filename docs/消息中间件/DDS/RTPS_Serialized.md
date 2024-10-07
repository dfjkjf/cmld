---
sort: 5
---

# 数据序列化

## 1. 序列化应用数据的位置及责任主体
- 位置
    - RTPS 协议在 SerializedPayload 子消息元素中传输序列化应用数据（见 9.4.2.12）。
- 责任主体
    - RTPS 协议本身不解释 SerializedPayload 的内容，只是将其作为不透明的字节集进行传递。对于使用 RTPS 的 DDS 来说，数据序列化和反序列化的责任分别在于 DDS DataWriter 和 DataReader。

## 2. 序列化相关组件
 - **SerializedPayloadHeader和Representation Identifier**
     - 所有的SerializedPayload都以SerializedPayloadHeader开始，它提供了关于后续数据表示的信息。SerializedPayloadHeader包含RepresentationOptions和RepresentationIdentifier两个部分。RepresentationIdentifier用于识别数据表示方式，RepresentationOptions需结合RepresentationIdentifier进行解释，且在某些情况下可能不被使用（如当前协议版本对于某些端点可能将其设为零并忽略）。为了对齐目的，CDR流在representation_options之后逻辑重置，避免在序列化数据前添加初始填充。
 - **不同类型端点的SerializedPayload格式**
     - **RTPS发现内置端点**：其SerializedPayload使用特定的RepresentationIdentifier值和格式。例如，对于某些情况使用PL_CDR_BE（ParameterList的OMG CDR Big Endian封装）或PL_CDR_LE（ParameterList的OMG CDR Little Endian封装），发送方需将representation_options设为零，接收方忽略该值。
     - **其他RTPS内置端点**：使用的RepresentationIdentifier值和格式与发现内置端点有所不同，可能包括CDR_BE（经典CDR Big Endian编码）、CDR_LE（经典CDR Little Endian编码）、PL_CDR_BE、PL_CDR_LE等，并且每个内置端点的定义应指示所使用的序列化数据格式和RepresentationIdentifier。
     - **用户定义的DDS主题**：其SerializedPayload依据DDS - XTYPES相关条款定义的数据表示来使用相应的RepresentationIdentifier值和格式，包括多种类型如CDR_BE、CDR_LE、PL_CDR_BE、PL_CDR_LE、CDR2_BE、CDR2_LE、PL_CDR2_BE、PL_CDR2_LE、D_CDR_BE、D_CDR_LE、XML等。对于不兼容DDS - XTYPES的遗留DDS实现，至少应支持CDR_BE和CDR_LE以及相关类型系统元素。

## 3. 示例说明
 - **内置端点数据示例**：以SEDPbuiltinSubscriptionsWriter声明一个DataReader为例，展示了SerializedPayload元素的布局和实际字节内容。包括数据读取器的相关信息（如主题、类型、端点GUID、目的地顺序、期限等）如何在SerializedPayload中体现，以及如何根据表示标识符（如PL_LE表示小端表示）进行序列化，其中包含了多个参数的编码和排列方式。
 - **用户定义主题数据示例**：以一个应用DataWriter发送关于特定主题和类型的数据为例，展示了在使用PLAIN_CDR表示（小端字节顺序）时SerializedPayload元素的布局和实际字节内容。包括数据值（如颜色、坐标、大小等）如何在SerializedPayload中进行序列化。

## 如何确保数据在序列化和反序列化过程中的一致性？

为确保数据在序列化和反序列化过程中的一致性，可采取以下措施：

### 1. 标准化的序列化头部和标识
 - **SerializedPayloadHeader的规范作用**
     - 所有的SerializedPayload都以SerializedPayloadHeader开始，它提供了关于后续数据表示的重要信息。其中的RepresentationIdentifier用于明确数据所使用的表示方式，不同的端点类型（如RTPS发现内置端点、其他RTPS内置端点、用户定义的DDS主题相关端点）都有其对应的规定值和格式。这种标准化的头部信息为数据的序列化和反序列化提供了统一的标识基础，确保发送方和接收方能够对数据的基本表示形式达成共识。
 - **RepresentationOptions的配合使用**
     - RepresentationOptions需结合RepresentationIdentifier进行解释，尽管在某些情况下（如当前协议版本对于某些端点）可能不被充分使用，但它也是数据表示规范的一部分。这种配合使用的机制有助于进一步细化数据表示的规则，在需要时可以提供更多关于数据序列化的信息，保证数据在不同场景下的一致性处理。

### 2. 明确的端点相关序列化规则
 - **针对不同端点的特定规则**
     - 对于RTPS发现内置端点，协议明确规定了其SerializedPayload应使用的RepresentationIdentifier值和格式（如PL_CDR_BE或PL_CDR_LE），并且发送方和接收方对representation_options的处理方式也有明确要求（发送方设为零，接收方忽略）。
     - 其他RTPS内置端点同样有各自适用的RepresentationIdentifier值和格式（如CDR_BE、CDR_LE等），且每个内置端点的定义都应指示所使用的序列化数据格式和RepresentationIdentifier，确保在这些端点上的数据序列化和反序列化操作遵循统一的规则。
     - 用户定义的DDS主题相关端点依据DDS - XTYPES相关条款定义的数据表示来确定RepresentationIdentifier值和格式，多种可能的格式（如CDR_BE、CDR_LE等）都有明确的使用场景和规则，使得在处理这类端点的数据时能够保持一致性。

### 3. 数据结构和参数编码的一致性
 - **数据结构的规范表示**
     - 在序列化过程中，数据结构中的各个元素按照规定的顺序和格式进行编码。例如，在给出的内置端点数据和用户定义主题数据的示例中，可以看到数据读取器的相关信息（如主题、类型、端点GUID等）以及应用数据的值（如颜色、坐标、大小等）都有特定的编码方式和排列顺序。这种规范的表示方式确保了数据在序列化后能够以相同的结构被反序列化，从而保证了数据的一致性。
 - **参数的准确编码和传递**
     - 对于一些参数（如在示例中出现的各种参数ID对应的参数），它们在SerializedPayload中的编码也是遵循一定规则的。这些参数的准确编码和传递确保了数据的完整性和一致性，接收方能够根据相同的规则正确地解析这些参数，还原出原始的数据信息。

### 4. 版本兼容性考虑
 - **对不同协议版本的兼容处理**
     - 在协议的发展过程中，考虑到不同版本的兼容性。例如，对于RTPS规范之前的版本可能存在的CDR流对齐问题（如版本2.4之前未明确CDR流在何处重置），在当前的序列化机制中进行了相应的规范和说明，要求实现过程中考虑供应商和协议版本来正确解释序列化数据，以确保不同版本之间数据序列化和反序列化的一致性。同时，对于参数ID空间的划分和保留情况也考虑了版本兼容性，避免因版本更新导致数据处理不一致的情况发生。