# NExT-GPT-Multimodal-LLM

While recently Multimodal Large Language Models (MM-LLMs) have made exciting strides, they mostly fall prey to the limitation of only input-side multimodal understanding, without the ability to produce content in multiple modalities. As we humans always perceive the world and communicate with people through various modalities, developing any-to-any MM-LLMs capable of accepting and delivering content in any modality becomes essential to human-level AI. To fill the gap, we present an end-to-end general-purpose any-to-any MM-LLM system, NExT-GPT. We connect an LLM with multimodal adaptors and different diffusion decoders, enabling NExT-GPT to perceive inputs and generate outputs in arbitrary combinations of text, images, videos, and audio. By leveraging the existing well-trained highly-performing encoders and decoders, NExT-GPT is tuned with only a small amount of parameter (1%) of certain projection layers, which not only benefits low-cost training and also facilitates convenient expansion to more potential modalities. Moreover, we introduce a modality-switching instruction tuning (MosIT) and manually curate a high-quality dataset for MosIT, based on which NExT-GPT is empowered with complex cross-modal semantic understanding and content generation. Overall, our research showcases the promising possibility of building an AI agent capable of modeling universal modalities, paving the way for more human-like AI research in the community.

![Alt text](images/Architecture.png)

There are three main tiers. 
## Multimodal Encoding Stage. 
Leveraging existing well-established models to encode inputs of various modalities. Here we take advantage of the ImageBind, which is a unified high-performance encoder across six modalities. Then, via the linear projection layer, different input representations are mapped into language-like representations that are comprehensible to the LLM.

## LLM Understanding and Reasoning Stage. 
An LLM is used as the core agent of NExT-GPT. Technically, we employ the Vicuna. LLM takes as input the representations from different modalities and carries out semantic understanding and reasoning over the inputs. It outputs 1) the textual responses directly, and 2) signal tokens of each modality that serve as instructions to dictate the decoding layers whether to generate multimodal contents, and what content to produce if yes.

## Multimodal Generation Stage. 
Receiving the multimodal signals with specific instructions from LLM (if any), the Transformer-based output projection layers map the signal token representations into the ones that are understandable to following multimodal decoders. Technically, we employ the current off-the-shelf latent conditioned diffusion models of different modal generations, i.e., Stable Diffusion (SD) for image synthesis, Zeroscope for video synthesis, and AudioLDM for audio synthesis.

![Alt text](images/parameters.png)

If LLM identifies a certain modality content (except language) to be produced, a special type of token will be output indicating the activation of that modality; otherwise, no special token output means deactivation of that modality. 

## Lightweight Multimodal Alignment Learning

We design the system with mainly three tiers in loose coupling, and we only need to update the two projection layers at encoding side and decoding side.

### Encoding-side LLM-centric Multimodal Alignment. 
We align different inputting multimodal features with the text feature space, the representations that are understandable to the core LLM. This is thus intuitively named the LLM-centric multimodal alignment learning. To accomplish the alignment, we prepare the ‘X-caption’ pair (‘X’ stands for image, audio, or video) data from existing corpus and benchmarks. We enforce LLM to produce the caption of each input modality against the gold caption. Figure 3(a) illustrates the learning process.

![Alt text](images/encoding.png)

### Decoding-side Instruction-following Alignment. 

On the decoding end, we have integrated pre-trained conditional diffusion models from external resources. Our main purpose is to align the diffusion models with LLM’s output instructions. However, performing a full-scale alignment process between each diffusion model and the LLM would entail a significant computational burden. Alternatively, we here explore a more efficient approach, decoding-side instruction-following alignment, as depicted in Figure 3(b). Specifically, since diffusion models of various modalities are conditioned solely on textual token inputs. This conditioning diverges significantly from the modal signal tokens from LLM in our system, which leads to a gap in the diffusion models’ accurate interpretation of the instructions from LLM. Thus, we consider minimizing the distance between the LLM’s modal signal token representations (after each Transformer-based project layer) and the conditional text representations of the diffusion models. Since only the textual condition encoders are used (with the diffusion backbone frozen), the learning is merely based on the purely captioning texts, i.e., without any visual or audio resources. This also ensures highly lightweight training.

![Alt text](images/decoding.png)
