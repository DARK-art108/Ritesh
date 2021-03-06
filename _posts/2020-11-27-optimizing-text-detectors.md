---
keywords: fastai
description: "This post compares two Deep Learning-based text detectors CRAFT and EAST with respect to deployment-specific requirements."
title: "A Battle of Text Detectors for Mobile Deployments: CRAFT vs. EAST"
toc: true
branch: master
badges: false
image: images/text_detector_benchmark.png
comments: true
author: Sayak Paul & Tulasi Ram Laghumavarapu
permalink: /optimizing-text-detectors/
categories: [tflite, model-optimization, text-detection]
nb_path: _notebooks/2020-11-27-optimizing-text-detectors.ipynb
layout: notebook
---

<!--
#################################################
### THIS FILE WAS AUTOGENERATED! DO NOT EDIT! ###
#################################################
# file to edit: _notebooks/2020-11-27-optimizing-text-detectors.ipynb
-->

<div class="container" id="notebook-container">
        
    {% raw %}
    
<div class="cell border-box-sizing code_cell rendered">

</div>
    {% endraw %}

<div class="cell border-box-sizing text_cell rendered"><div class="inner_cell">
<div class="text_cell_render border-box-sizing rendered_html">
<p>In the <a href="https://tulasi.dev/craft-in-tflite">previous post</a>, we saw how to convert the pre-trained <a href="https://arxiv.org/pdf/1904.01941">CRAFT</a> model from PyTorch to TensorFlow Lite (TFLite) and run inference with the converted TFLite model. In this post, we will be comparing the TFLite variants of the CRAFT model to another text detection model - <a href="https://arxiv.org/abs/1704.03155">EAST</a>. The objective of this post is to provide a comparative study between these two models with respect to various deployment-specific pointers such as inference latency, model size, performance on dense text regions, and so on. Text detection continues to be a very important use-case across many verticals. So we hope this post will serve as a systematic guide for developers that are interested to explore on-device text detection models.</p>
<p>Precisely, we will be comparing the two models on the basis of the following pointers which we think are very crucial when it comes to deploying them out in the wild -</p>
<ul>
<li>Visual Inspection of Performance</li>
<li>Model Size</li>
<li>Inference Latency</li>
<li>Memory Usage</li>
</ul>

</div>
</div>
</div>
<div class="cell border-box-sizing text_cell rendered"><div class="inner_cell">
<div class="text_cell_render border-box-sizing rendered_html">
<p>{% include important.html content='If you are interested to know about the conversion process and inference pipelines of the models, please refer to these notebooks - <a href="https://github.com/tulasiram58827/craft_tflite/tree/main/colabs">CRAFT</a> and <a href="https://github.com/sayakpaul/Adventures-in-TensorFlow-Lite/blob/master/EAST_TFLite.ipynb">EAST</a>. The pre-converted models are available on TensorFlow Hub - <a href="https://tfhub.dev/tulasiram58827/lite-model/craft-text-detector/dr/1">CRAFT</a> and <a href="https://tfhub.dev/sayakpaul/lite-model/east-text-detector/dr/1">EAST</a>.' %}</p>

</div>
</div>
</div>
<div class="cell border-box-sizing text_cell rendered"><div class="inner_cell">
<div class="text_cell_render border-box-sizing rendered_html">
<h2 id="Benchmark-Setup">Benchmark Setup<a class="anchor-link" href="#Benchmark-Setup"> </a></h2><p>We used the <a href="https://www.tensorflow.org/lite/performance/measurement">TensorFlow Lite Benchmark tool</a> in order to gather results on inference latency and memory usage of the models with <strong>Redmi K20 Pro</strong> as the target device. We chose a mobile device for this purpose because text detection is a pretty prevalent recipe of many mobile applications such as <a href="https://play.google.com/store/apps/details?id=com.google.ar.lens&amp;hl=en_IN&amp;gl=US">Google Lens</a>.</p>
<p>In order to make the comparisons fair, we consider the two models with three different image resolutions - 320x320, 640x416, and 1200x800. For each of these resolutions, we consider two different <a href="https://www.tensorflow.org/lite/performance/post_training_quantization">post-training quantization schemes</a> - dynamic-range and float16. <em>The CRAFT model conversion is not yet supported in the integer variant, hence we do not consider integer quantization (but the EAST model does support it)</em>.</p>

</div>
</div>
</div>
<div class="cell border-box-sizing text_cell rendered"><div class="inner_cell">
<div class="text_cell_render border-box-sizing rendered_html">
<h2 id="Visual-Inspection-of-Performance">Visual Inspection of Performance<a class="anchor-link" href="#Visual-Inspection-of-Performance"> </a></h2><p>In this setting, we run both of the models and their different variants (dynamic-range and float16 quantized) on a sample image that has dense text regions, and then we visualize the results. We observed that both of these models perform fairly well on images having lighter text regions. Here???s the sample image we used for the purpose -</p>
<p><img src="https://i.ibb.co/KVKnnct/image.png" alt=""></p>
<center>
    <small>Image is taken from the <a href="https://rrc.cvc.uab.es/?ch=13">SROIE dataset</a>.</small><br>
</center><p>Time to detect some texts!</p>

</div>
</div>
</div>
<div class="cell border-box-sizing text_cell rendered"><div class="inner_cell">
<div class="text_cell_render border-box-sizing rendered_html">
<h3 id="CRAFT---320x320-Dynamic-Range-&amp;-float16">CRAFT - 320x320 Dynamic-Range &amp; float16<a class="anchor-link" href="#CRAFT---320x320-Dynamic-Range-&amp;-float16"> </a></h3><p>In the dynamic-range quantization setting, we can see the model misses out on some text blocks.</p>
<p><img src="https://i.ibb.co/RBX8XDn/image-w-593-h-442-rev-1-ac-1-parent-19-Qb-WABWc-E3n-SLPE6zqm-Tw6b-Vxkxee-YUw-Om-KTDn-Dz8k-Q.png" alt=""></p>
<center>
    <small>Inference results from the 320x320 dynamic-range and float16 quantized CRAFT models.</small><br>
</center><p>With increased numerical precision i.e. float16, we can clearly see quite a bit of improvement in the results. It???s important to note that this improvement comes at the cost of increased model size.</p>
<p>Next up, we apply the same steps to the EAST model.</p>
<h3 id="EAST---320x320-Dynamic-Range-&amp;-float16">EAST - 320x320 Dynamic-Range &amp; float16<a class="anchor-link" href="#EAST---320x320-Dynamic-Range-&amp;-float16"> </a></h3><p>EAST apparently performs better than CRAFT under dynamic-range quantization. If we look closely, it appears that the CRAFT model produces far fewer overlaps in the detections compared to EAST. On the other hand, the EAST model is able to detect more text blocks. When developing practical applications with text detectors, it often becomes a classic case of <em>precision-recall</em> trade-offs like the one we are currently seeing. So, you would want to consider the application-specific needs in order to decide the level of trade-off to be achieved there.</p>
<p><img src="https://i.ibb.co/qsCMC5N/image-w-624-h-520-rev-37-ac-1-parent-19-Qb-WABWc-E3n-SLPE6zqm-Tw6b-Vxkxee-YUw-Om-KTDn-Dz8k-Q.png" alt=""></p>
<center>
    <small>Inference results from the 320x320 dynamic-range and float16 quantized EAST models.</small><br>
</center><p>With increased precision, the above-mentioned points still hold, i.e. the number of overlaps being way higher for the EAST model than they are in the CRAFT equivalent. In this setting (float16 quantization), superiority in the performance of the CRAFT model is quite evident in regards to the EAST model.</p>
<p>As different applications may use different image resolutions we decided to test the performance of the models on larger dimensions as well. This is what we are going to see next.</p>

</div>
</div>
</div>
<div class="cell border-box-sizing text_cell rendered"><div class="inner_cell">
<div class="text_cell_render border-box-sizing rendered_html">
<h3 id="CRAFT---640x416-Dynamic-Range-&amp;-float16">CRAFT - 640x416 Dynamic-Range &amp; float16<a class="anchor-link" href="#CRAFT---640x416-Dynamic-Range-&amp;-float16"> </a></h3><p>On an increased resolution, the CRAFT model performs pretty well -</p>
<p><img src="https://i.ibb.co/VxbyWch/image-w-624-h-568-rev-38-ac-1-parent-19-Qb-WABWc-E3n-SLPE6zqm-Tw6b-Vxkxee-YUw-Om-KTDn-Dz8k-Q.png" alt=""></p>
<center>
    <small>Inference results from the 640x416 dynamic-range and float16 quantized CRAFT models.</small><br>
</center><p>The float16 version of this resolution is a slam dunk (rightfully leaving behind the barcode which is not a piece of text).</p>
<h3 id="EAST---640x416-Dynamic-Range-&amp;-float16">EAST - 640x416 Dynamic-Range &amp; float16<a class="anchor-link" href="#EAST---640x416-Dynamic-Range-&amp;-float16"> </a></h3><p>The performance of the EAST model under these settings are very equivalent to CRAFT -</p>
<p><img src="https://i.ibb.co/ynBbrFZ/image-w-597-h-612-rev-36-ac-1-parent-19-Qb-WABWc-E3n-SLPE6zqm-Tw6b-Vxkxee-YUw-Om-KTDn-Dz8k-Q.png" alt=""></p>
<center>
    <small>Inference results from the 640x416 dynamic-range and float16 quantized EAST models.</small><br>
</center><p>With float16 quantization and 640x416 as the resolution, the CRAFT model is a clear winner. Notice that the EAST model is still unable to discard the barcode part which might be an important point to note for some applications.</p>
<p>Time to inspect the results for our final and highest resolution - 1280x800.</p>

</div>
</div>
</div>
<div class="cell border-box-sizing text_cell rendered"><div class="inner_cell">
<div class="text_cell_render border-box-sizing rendered_html">
<h3 id="CRAFT---1280x800-Dynamic-Range-&amp;-float16">CRAFT - 1280x800 Dynamic-Range &amp; float16<a class="anchor-link" href="#CRAFT---1280x800-Dynamic-Range-&amp;-float16"> </a></h3><p>Under dynamic-range quantization, the results look okayish. The model misses out on a number of text blocks but the only ones that it detects appear to be neat.</p>
<p><img src="https://i.ibb.co/QMDpH9M/image-w-624-h-453-rev-34-ac-1-parent-19-Qb-WABWc-E3n-SLPE6zqm-Tw6b-Vxkxee-YUw-Om-KTDn-Dz8k-Q.png" alt=""></p>
<center>
    <small>Inference results from the 1280x800 dynamic-range and float16 quantized CRAFT models.</small><br>
</center><p>The results from the float16 variant are tremendous (as you probably have guessed by now).</p>
<h3 id="EAST---1280x800-Dynamic-Range-&amp;-float16">EAST - 1280x800 Dynamic-Range &amp; float16<a class="anchor-link" href="#EAST---1280x800-Dynamic-Range-&amp;-float16"> </a></h3><p>At this resolution, the EAST model seems to be performing well too -</p>
<p><img src="https://i.ibb.co/xYHfXXn/image-w-624-h-483-rev-29-ac-1-parent-19-Qb-WABWc-E3n-SLPE6zqm-Tw6b-Vxkxee-YUw-Om-KTDn-Dz8k-Q.png" alt=""></p>
<center>
    <small>Inference results from the 1280x800 dynamic-range and float16 quantized EAST models.</small><br>
</center><p>With float16 quantization as well, the CRAFT model beats EAST in terms of the detection quality.</p>

</div>
</div>
</div>
<div class="cell border-box-sizing text_cell rendered"><div class="inner_cell">
<div class="text_cell_render border-box-sizing rendered_html">
<h2 id="Model-Size">Model Size<a class="anchor-link" href="#Model-Size"> </a></h2><p>When it comes to deploying models to mobile devices model size becomes a really important factor. You may not want to have a heavy model that would, in turn, make your mobile application bulky. Moreover, <a href="https://support.google.com/googleplay/android-developer/answer/113469#apk">Playstore</a> and <a href="https://developer.apple.com/forums/thread/12455">AppStore</a> also have size restrictions on the applications one can host there.</p>
<p>On the other hand, heavier models tend to be slower. If your application cannot have increased inference latency then you would want to have the model size as low as possible.</p>
<p>The following figure shows the size of the CRAFT and EAST models -</p>
<p><img src="https://i.ibb.co/tX7bknk/nyrm-wh-z-itikr9-cnyl6-z1-fq3.png" alt=""></p>
<center>
    <small>Model (TFLite variants) sizes of CRAFT and EAST.</small><br>
</center><p>The dynamic-range quantized versions of both the models are in a well-acceptable range with respect to size. However, the float16 variants may still be a bit heavier for some applications.</p>

</div>
</div>
</div>
<div class="cell border-box-sizing text_cell rendered"><div class="inner_cell">
<div class="text_cell_render border-box-sizing rendered_html">
<h2 id="Inference-Latency">Inference Latency<a class="anchor-link" href="#Inference-Latency"> </a></h2><p>Inference latency is also one of the major factors for mobile-based deployments especially when your applications might require instantaneous predictions. We are going to show a comparison between all the settings we considered in the visual inspection section.</p>
<p>To reiterate we performed the benchmarks for this section on a Redmi K20 Pro using 4 threads. In the following figures, we present inference latency of different variants of the CRAFT and EAST models.</p>
<p><img src="https://i.ibb.co/1GyPgR6/ylz3-vh2l-ownf4av-amai-w0j-oz.png" alt=""></p>
<center>
    <small>Inference latency of different variants of the CRAFT model.</small><br>
</center><p><img src="https://i.ibb.co/ySBsQvs/z-q-o-zf7cl-hu-tfh-ou-a7-yscgm.png" alt=""></p>
<center>
    <small>Inference latency of different variants of the EAST model.</small><br>
</center><p>As expected, with increased resolution the inference latency also increases. Inference latency is also quite lower for all the variants of the EAST model compared to CRAFT. Earlier we saw how a quantization affects model performance under a particular resolution. As stated earlier, when using these models inside a mobile application, the ???<em>Size vs. Performance</em>??? trade-off becomes extremely vital. 
{% include important.html content='The results for the float16 1280x800 CRAFT model could not be obtained on our target device.' %}</p>

</div>
</div>
</div>
<div class="cell border-box-sizing text_cell rendered"><div class="inner_cell">
<div class="text_cell_render border-box-sizing rendered_html">
<h2 id="Memory-Usage">Memory Usage<a class="anchor-link" href="#Memory-Usage"> </a></h2><p>In section, we shed light on the total memory allocated for the models while running the TensorFlow Lite Benchmark tool. Knowing about the memory usage of these models helps us plan application releases accordingly as not all the mobile phones may support extensive memory requirements. So based on this information, you may want to set some device requirements for your application using these models. On the other hand, if you would want your application to be as device-agnostic as possible then you may want to maintain separate models according to their size and memory usage.</p>
<p>In this case, also, we are going to consider all the settings we had considered in the previous sections. The following figures give us a sense of the memory footprint left behind by the models -</p>
<p><img src="https://i.ibb.co/TrnZ9vX/webp-net-resizeimage.png" alt=""></p>
<center>
    <small>Memory footprint of different variants of the CRAFT model.</small><br>
</center><p><img src="https://i.ibb.co/3szkpK0/hfp-jmc4-nej-lloj-bc2-q-nz515y.png" alt=""></p>
<center>
    <small>Memory footprint of different variants of the EAST model.</small><br>
</center><p>Detection performance-wise, CRAFT was a winner in many cases but if we factor in for inference latency and memory footprint the situation might need reconsideration. In other words, the best performing (with respect to a certain task, detection in this case) model may not always be the best candidate for deployments. 
{% include important.html content='The results for the float16 1280x800 CRAFT model could not be obtained on our target device.' %}</p>

</div>
</div>
</div>
<div class="cell border-box-sizing text_cell rendered"><div class="inner_cell">
<div class="text_cell_render border-box-sizing rendered_html">
<h2 id="Conclusion">Conclusion<a class="anchor-link" href="#Conclusion"> </a></h2><p>In this post, we presented a comparative study between two text detection models - CRAFT and EAST.  We went beyond their task-specific performance and considered various essential factors that one needs to consider when deploying these models. At this point, you might have felt the need to consider another important factor of these models - <em>FPS information of the models on real-time videos</em>. Please check out <a href="https://github.com/farmaker47/OCR_with_Keras">this repository</a> to get a handle on how to approach that development.</p>
<h2 id="Contribution">Contribution<a class="anchor-link" href="#Contribution"> </a></h2><p><a href="https://www.linkedin.com/in/tulasi-ram-laghumavarapu-aba672103/">Tulasi</a> worked on the CRAFT model while Sayak worked on the EAST model. For the purpose of this post, Tulasi focused on gathering all the relevant information for doing the comparisons while Sayak focused on the writing part.</p>
<p>Thanks to <a href="https://twitter.com/khanhlvg">Khanh LeViet</a> from the TFLite team for reviewing the post.</p>

</div>
</div>
</div>
</div>
 

