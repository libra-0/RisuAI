<script lang="ts">
    import Check from "src/lib/UI/GUI/CheckInput.svelte";
    import { language } from "src/lang";
    import Help from "src/lib/Others/Help.svelte";
    import { selectSingleFile } from "src/ts/util";
    import { DataBase } from "src/ts/storage/database";
    import { isTauri, saveAsset } from "src/ts/storage/globalApi";
    import NumberInput from "src/lib/UI/GUI/NumberInput.svelte";
    import TextInput from "src/lib/UI/GUI/TextInput.svelte";
    import SelectInput from "src/lib/UI/GUI/SelectInput.svelte";
    import OptionInput from "src/lib/UI/GUI/OptionInput.svelte";
    import SliderInput from "src/lib/UI/GUI/SliderInput.svelte";
    import Button from "src/lib/UI/GUI/Button.svelte";
    import { getCharImage } from "src/ts/characters";
    import Arcodion from "src/lib/UI/Arcodion.svelte";

</script>
<h2 class="mb-2 text-2xl font-bold mt-2">{language.otherBots}</h2>


<Arcodion name={language.imageGeneration}  styled>
    <span class="text-textcolor mt-2">{language.imageGeneration} {language.provider} <Help key="sdProvider"/></span>
    <SelectInput className="mt-2 mb-4" bind:value={$DataBase.sdProvider}>
        <OptionInput value="" >None</OptionInput>
        <OptionInput value="webui" >Stable Diffusion WebUI</OptionInput>
        <OptionInput value="novelai" >Novel AI</OptionInput>
        <!-- TODO -->
        <!-- <OptionInput value="runpod" >Runpod Serverless</OptionInput> -->
    </SelectInput>
    
    {#if $DataBase.sdProvider === 'webui'}
    <span class="text-draculared text-xs mb-2">You must use WebUI with --api flag</span>
        <span class="text-draculared text-xs mb-2">You must use WebUI without agpl license or use unmodified version with agpl license to observe the contents of the agpl license.</span>
        {#if !isTauri}
            <span class="text-draculared text-xs mb-2">You are using web version. you must use ngrok or other tunnels to use your local webui.</span>
        {/if}
        <span class="text-textcolor mt-2">WebUI {language.providerURL}</span>
        <TextInput size="sm" marginBottom placeholder="https://..." bind:value={$DataBase.webUiUrl}/>
        <span class="text-textcolor">Steps</span>
        <NumberInput size="sm" marginBottom min={0} max={100} bind:value={$DataBase.sdSteps}/>
        
        <span class="text-textcolor">CFG Scale</span>
        <NumberInput size="sm" marginBottom min={0} max={20} bind:value={$DataBase.sdCFG}/>
    
        <span class="text-textcolor">Width</span>
        <NumberInput size="sm" marginBottom min={0} max={2048} bind:value={$DataBase.sdConfig.width}/>
        <span class="text-textcolor">Height</span>
        <NumberInput size="sm" marginBottom min={0} max={2048} bind:value={$DataBase.sdConfig.height}/>
        <span class="text-textcolor">Sampler</span>
        <TextInput size="sm" marginBottom bind:value={$DataBase.sdConfig.sampler_name}/>
        
        <div class="flex items-center mt-2">
            <Check bind:check={$DataBase.sdConfig.enable_hr} name='Enable Hires'/>
        </div>
        {#if $DataBase.sdConfig.enable_hr === true}
            <span class="text-textcolor">denoising_strength</span>
            <NumberInput size="sm" marginBottom  min={0} max={10} bind:value={$DataBase.sdConfig.denoising_strength}/>
            <span class="text-textcolor">hr_scale</span>
            <NumberInput size="sm" marginBottom  min={0} max={10} bind:value={$DataBase.sdConfig.hr_scale}/>
            <span class="text-textcolor">Upscaler</span>
            <TextInput size="sm" marginBottom bind:value={$DataBase.sdConfig.hr_upscaler}/>
        {/if}
    {/if}
    
    {#if $DataBase.sdProvider === 'novelai'}
        <span class="text-textcolor mt-2">Novel AI {language.providerURL}</span>
        <TextInput size="sm" marginBottom placeholder="https://api.novelai.net" bind:value={$DataBase.NAIImgUrl}/>
        <span class="text-textcolor">API Key</span>
        <TextInput size="sm" marginBottom placeholder="pst-..." bind:value={$DataBase.NAIApiKey}/>
    
        <span class="text-textcolor">Model</span>
        <TextInput size="sm" marginBottom placeholder="nai-diffusion-3" bind:value={$DataBase.NAIImgModel}/>
    
        <span class="text-textcolor">Width</span>
        <NumberInput size="sm" marginBottom min={0} max={2048} bind:value={$DataBase.NAIImgConfig.width}/>
        <span class="text-textcolor">Height</span>
        <NumberInput size="sm" marginBottom min={0} max={2048} bind:value={$DataBase.NAIImgConfig.height}/>
        <span class="text-textcolor">Sampler</span>
        <TextInput size="sm" marginBottom bind:value={$DataBase.NAIImgConfig.sampler}/>
        <span class="text-textcolor">steps</span>
        <NumberInput size="sm" marginBottom min={0} max={2048} bind:value={$DataBase.NAIImgConfig.steps}/>
        <span class="text-textcolor">CFG scale</span>
        <NumberInput size="sm" marginBottom min={0} max={2048} bind:value={$DataBase.NAIImgConfig.scale}/>
    
        {#if !$DataBase.NAII2I}
            <Check bind:check={$DataBase.NAIImgConfig.sm} name="Use SMEA"/>
            <Check bind:check={$DataBase.NAIImgConfig.sm_dyn} name='Use DYN'/>
        {/if}
        <Check bind:check={$DataBase.NAII2I} name="Enable I2I"/>
    
        {#if $DataBase.NAII2I}
    
            <span class="text-textcolor mt-4">strength</span>
            <SliderInput min={0} max={0.99} step={0.01} bind:value={$DataBase.NAIImgConfig.strength}/>
            <span class="text-textcolor2 mb-6 text-sm">{$DataBase.NAIImgConfig.strength}</span>
            <span class="text-textcolor">noise</span>
            <SliderInput min={0} max={0.99} step={0.01} bind:value={$DataBase.NAIImgConfig.noise}/>
            <span class="text-textcolor2 mb-6 text-sm">{$DataBase.NAIImgConfig.noise}</span>
    
            <span class="text-textcolor">Base image</span>
            <button on:click={async () => {
                const img = await selectSingleFile([
                    'jpg',
                    'jpeg',
                    'png',
                    'webp'
                ])
                if(!img){
                    return null
                }
                const saveId = await saveAsset(img.data)
                $DataBase.NAIImgConfig.image = saveId
            }}>
                {#if $DataBase.NAIImgConfig.image === ''}
                    <div class="rounded-md h-20 w-20 shadow-lg bg-textcolor2 cursor-pointer hover:text-green-500"/>
                {:else}
                    {#await getCharImage($DataBase.NAIImgConfig.image, 'css')}
                        <div class="rounded-md h-20 w-20 shadow-lg bg-textcolor2 cursor-pointer hover:text-green-500"/>
                    {:then im} 
                        <div class="rounded-md h-20 w-20 shadow-lg bg-textcolor2 cursor-pointer hover:text-green-500" style={im} />                
                    {/await}
                {/if}
            </button>
    
        {/if}
    {/if}
</Arcodion>

<Arcodion name="TTS" styled>
    <span class="text-textcolor mt-2">ElevenLabs API key</span>
    <TextInput size="sm" marginBottom bind:value={$DataBase.elevenLabKey}/>
    
    <span class="text-textcolor mt-2">VOICEVOX URL</span>
    <TextInput size="sm" marginBottom bind:value={$DataBase.voicevoxUrl}/>
    
    <span class="text-textcolor mt-2">NovelAI API key</span>
    <TextInput size="sm" marginBottom placeholder="pst-..." bind:value={$DataBase.NAIApiKey}/>
    
    
</Arcodion>

<Arcodion name={language.emotionImage} styled>
    <span class="text-textcolor mt-2">{language.emotionMethod}</span>

    <SelectInput className="mt-2 mb-4" bind:value={$DataBase.emotionProcesser}>
        <OptionInput value="submodel" >Ax. Model</OptionInput>
        <OptionInput value="embedding" >MiniLM-L6-v2</OptionInput>
    </SelectInput>
</Arcodion>

<Arcodion name="superMemory" styled>
    <span class="text-textcolor mt-4">{language.SuperMemory} {language.model}</span>
    <SelectInput className="mt-2 mb-2" bind:value={$DataBase.supaMemoryType}>
        <OptionInput value="none" >None</OptionInput>
        <OptionInput value="distilbart" >distilbart-cnn-6-6 (Free/Local)</OptionInput>
        <OptionInput value="instruct35" >OpenAI 3.5 Turbo Instruct</OptionInput>
        <OptionInput value="davinci" >OpenAI Davinci</OptionInput>
        <OptionInput value="curie" >OpenAI Curie</OptionInput>
        <OptionInput value="subModel" >{language.submodel} ({language.unrecommended})</OptionInput>
    </SelectInput>
    {#if $DataBase.supaMemoryType === 'davinci' || $DataBase.supaMemoryType === 'curie' || $DataBase.supaMemoryType === 'instruct35'}
        <span class="text-textcolor">{language.SuperMemory} OpenAI Key</span>
        <TextInput size="sm" marginBottom bind:value={$DataBase.supaMemoryKey}/>
    {/if}
    {#if $DataBase.supaMemoryType !== 'none'}
        <span class="text-textcolor">{language.SuperMemory} Prompt</span>
        <TextInput size="sm" marginBottom bind:value={$DataBase.supaMemoryPrompt} placeholder="recommended to leave it blank to use default"/>
    {/if}
    {#if $DataBase.hypaMemory}
        <span class="text-textcolor">{language.HypaMemory} Model</span>
        <SelectInput className="mt-2 mb-2" bind:value={$DataBase.hypaModel}>
            <OptionInput value="MiniLM" >MiniLM-L6-v2 (Free / Local)</OptionInput>
            <OptionInput value="ada" >OpenAI Ada (Davinci / Curie Only)</OptionInput>
        </SelectInput>
    {/if}
    {#if $DataBase.useExperimental}
        <div class="flex">
            <Check bind:check={$DataBase.hypaMemory} name={language.able + ' ' + language.HypaMemory}/> <Help key="experimental" />
        </div>
    {/if}
</Arcodion>