<script lang="ts">
    import Check from "src/lib/UI/GUI/CheckInput.svelte";
    import { changeLanguage, language } from "src/lang";
    import { DataBase } from "src/ts/storage/database";
    import { sleep } from "src/ts/util";
    import Help from "src/lib/Others/Help.svelte";
    import OptionInput from "src/lib/UI/GUI/OptionInput.svelte";
    import SelectInput from "src/lib/UI/GUI/SelectInput.svelte";
    import { alertNormal } from "src/ts/alert";
    import { downloadFile } from "src/ts/storage/globalApi";
    import { languageEnglish } from "src/lang/en";
  import TextInput from "src/lib/UI/GUI/TextInput.svelte";
    let langChanged = false
</script>
<h2 class="mb-2 text-2xl font-bold mt-2">{language.language}</h2>

<span class="text-textcolor mt-4">{language.UiLanguage}</span>
<SelectInput className="mt-2" bind:value={$DataBase.language} on:change={async () => {
    if($DataBase.language === 'translang'){
        downloadFile('lang.json', new TextEncoder().encode(JSON.stringify(languageEnglish, null, 4)))
        alertNormal("Downloaded JSON, translate it, and send it to the dev by discord DM and email. I will add it to the next version.")
        $DataBase.language = 'en'
    }
    await sleep(10)
    changeLanguage($DataBase.language)
    langChanged = true
}}>
    <OptionInput value="de" >Deutsch</OptionInput>
    <OptionInput value="en" >English</OptionInput>
    <OptionInput value="ko" >한국어</OptionInput>
    <OptionInput value="cn" >中文</OptionInput>
    <OptionInput value="vi" >Tiếng Việt</OptionInput>
    <OptionInput value="translang" >[Translate in your own language]</OptionInput>
</SelectInput>
{#if langChanged}
    <span class="bg-red-500 text-sm">Close the settings to take effect</span>
{/if}
<span class="text-textcolor mt-4">{language.translatorLanguage}</span>
<SelectInput className="mt-2 mb-4" bind:value={$DataBase.translator}>
    <OptionInput value="" >{language.disabled}</OptionInput>
    <OptionInput value="ko" >Korean</OptionInput>
    <OptionInput value="ru" >Russian</OptionInput>
    <OptionInput value="zh" >Chinese</OptionInput>
    <OptionInput value="ja" >Japanese</OptionInput>
    <OptionInput value="fr" >French</OptionInput>
    <OptionInput value="es" >Spanish</OptionInput>
    <OptionInput value="pt" >Portuguese</OptionInput>
    <OptionInput value="de" >German</OptionInput>
    <OptionInput value="id" >Indonesian</OptionInput>
    <OptionInput value="ms" >Malaysian</OptionInput>
    <OptionInput value="uk" >Ukranian</OptionInput>
</SelectInput>

{#if $DataBase.translator}
    <span class="text-textcolor mt-4">{language.translatorType}</span>
    <SelectInput className="mt-2 mb-4" bind:value={$DataBase.translatorType}>
        <OptionInput value="google" >Google</OptionInput>
        <OptionInput value="deepl" >DeepL</OptionInput>
        <OptionInput value="LLM" >LLM</OptionInput>
    </SelectInput>

    {#if $DataBase.translatorType === 'deepl'}

        <span class="text-textcolor mt-4">{language.deeplKey}</span>
        <TextInput bind:value={$DataBase.deeplOptions.key} />

        <div class="flex items-center mt-2">
            <Check bind:check={$DataBase.deeplOptions.freeApi} name={language.deeplFreeKey}/>
        </div>

    {/if}

    {#if $DataBase.translatorType === 'LLM'}

        <span class="text-textcolor mt-4">{language.llmprompt}</span>
        <TextInput bind:value={$DataBase.llmoption.prompt} />

        <span class="text-textcolor mt-4">{language.llmserver}</span>
        <TextInput bind:value={$DataBase.llmoption.server} />

        <span class="text-textcolor mt-4">{language.llmkey}</span>
        <TextInput bind:value={$DataBase.llmoption.key} />

    {/if}


    <div class="flex items-center mt-2">
        <Check bind:check={$DataBase.autoTranslate} name={language.autoTranslation}/>
    </div>
{/if}