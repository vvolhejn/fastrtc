<style>
.tag-button {
    cursor: pointer;
    opacity: 0.5;
    transition: opacity 0.2s ease;
}

.tag-button > code {
    color: var(--supernova);
}

.tag-button.active {
    opacity: 1;
}
</style>

A collection of VAD models ready to use with FastRTC. Click on the tags below to find the VAD model you're looking for!


<div class="tag-buttons">
  <button class="tag-button" data-tag="pytorch"><code>pytorch</code></button>
</div>

<script>
function filterCards() {
    const activeButtons = document.querySelectorAll('.tag-button.active');
    const selectedTags = Array.from(activeButtons).map(button => button.getAttribute('data-tag'));
    const cards = document.querySelectorAll('.grid.cards > ul > li > p[data-tags]');
    
    cards.forEach(card => {
        const cardTags = card.getAttribute('data-tags').split(',');
        const shouldShow = selectedTags.length === 0 || selectedTags.some(tag => cardTags.includes(tag));
        card.parentElement.style.display = shouldShow ? 'block' : 'none';
    });
}
document.querySelectorAll('.tag-button').forEach(button => {
    button.addEventListener('click', () => {
        button.classList.toggle('active');
        filterCards();
    });
});
</script>


<div class="grid cards" markdown>

-   :speaking_head:{ .lg .middle }:eyes:{ .lg .middle } __Your VAD Model__
{: data-tags="pytorch"}

    ---

    Description

    Install Instructions

    Usage

    [:octicons-arrow-right-24: Demo](Your demo here)
    
    [:octicons-code-16: Repository](Code here)

</div>

## How to add your own VAD model

1. Your model can be implemented in **any** framework you want but it must implement the `PauseDetectionModel` protocol.
    ```python
    ModelOptions: TypeAlias = Any


    class PauseDetectionModel(Protocol):
        def vad(
            self,
            audio: tuple[int, NDArray[np.int16] | NDArray[np.float32]],
            options: ModelOptions,
        ) -> tuple[float, list[AudioChunk]]: ...

        def warmup(
            self,
        ) -> None: ...
    ```

    * The `vad` method should take a numpy array of audio data and return a tuple of the form `(speech_duration, and list[AudioChunk])` where `speech_duration` is the duration of the human speech in the audio chunk and `AudioChunk` is a dictionary with the following fields: `(start, end)` where `start` and `end` are the start and end times of the human speech in the audio array.

    * The `audio` tuple should be of the form `(sample_rate, audio_array)` where `sample_rate` is the sample rate of the audio array and `audio_array` is a numpy array of the audio data. It can be of type `np.int16` or `np.float32`.

    * The `warmup` method is optional but recommended to warm up the model when the server starts.

2. Once you have your model implemented, you can use it in the `ReplyOnPause` class by passing in the model and any options you need.

    ```python
    from fastrtc import ReplyOnPause, Stream
    from your_model import YourModel

    def echo(audio):
        yield audio

    model = YourModel() # implement the PauseDetectionModel protocol
    reply_on_pause = ReplyOnPause(
        echo,
        model=model,
        options=YourModelOptions(),
    )
    stream = Stream(reply_on_pause, mode="send-receive", modality="audio")
    stream.ui.launch()
    ```

3. Open a [PR](https://github.com/freddyaboulton/fastrtc/edit/main/docs/vad_gallery.md) to add your model to the gallery! Ideally you model package should be pip installable so other can try it out easily.