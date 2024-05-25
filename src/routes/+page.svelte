<script>
    import "carbon-components-svelte/css/g80.css";

    import {
        Header,
        Search,
        Content,
        Grid,
        Row,
        Column,
        Button,
        Tabs, Tab, TabContent, 
        DataTable,
        Slider,
        Tile,
    } from "carbon-components-svelte";
    import Play from "carbon-icons-svelte/lib/Play.svelte";
    import Stop from "carbon-icons-svelte/lib/Stop.svelte";
    import Pause from "carbon-icons-svelte/lib/Pause.svelte";

    let value = "";
    let musics = [];
    let albums = [];
    let folders = [];
    let libs = [];
    let min = 0;
    let max = 0;
    let nowtime = 0;
    let nowfile = 'None';

    async function getMusic() {
        let params = new URLSearchParams();
        params.append('search', value);

        fetch('/musics.json', {
            method: 'POST',
            body: params
        }).then(async function(res) {
            let json = await res.json();
            let cnt = 0;
            for(let l of json) {
                l.id = cnt;
                cnt++;
            }
            musics = json;
        });

        params.append('group', 'album');
        fetch('/musics.json', {
            method: 'POST',
            body: params
        }).then(async function(res) {
            let json = await res.json();
            let cnt = 0;
            for(let l of json) {
                l.id = cnt;
                cnt++;
            }
            albums = json;
        });

        params.append('group', 'folder');
        fetch('/musics.json', {
            method: 'POST',
            body: params
        }).then(async function(res) {
            let json = await res.json();
            let cnt = 0;
            for(let l of json) {
                l.id = cnt;
                cnt++;
            }
            folders = json;
        });
    }

    async function playMusic(row) {
        let params = new URLSearchParams();
        params.append('path', row.detail.path);
        params.append('name', row.detail.name);

        fetch('/play.json', {
            method: 'POST',
            body: params
        });
    }

    async function playFolder(row) {
        let params = new URLSearchParams();
        params.append('path', row.detail.path);

        fetch('/play.json', {
            method: 'POST',
            body: params
        });
    }

    async function getLib() {
        let res = await fetch('/lib.json', {
            method: 'POST'
        });
        libs = await res.json();
    }

    async function resumeMusic() {
        fetch('/resume.json');
    }

    async function stopMusic() {
        fetch('/stop.json');
    }

    async function pauseMusic() {
        fetch('/togglepause.json');
    }

    setTimeout(async function() {
       $: console.log('Init');
        //getLib();
        //getMusic();
        //streamStatus();
    }, 1000);

    function streamStatus() {
        let stream = new EventSource('/stream');
        stream.addEventListener('message', function(e) {
            if(!e.data) { return; }
            let st = '';
            try {
                st = JSON.parse(e.data);
                max = Math.ceil(st.duration);//Math.floor(st.duration/60) + ':' + Math.floor(st.duration%60);
                nowtime = Math.ceil(st.timePos);
                nowfile = st.title || st.file;
            } catch(e) {
                st = 0;
                max = 0;
                nowtime = 0;
                nowfile = 'None';
            }
        });
    }
</script>

<Header company="MONOBOX" platformName="Sowver">
</Header>

<Content>
  <Grid padding>
    <Row>
        <Column>
            <Search bind:value on:change={getMusic}/>
        </Column>
    </Row>
    <Row>
      <Column>
        <Tabs>
        <Tab label="Music Lists" />
        <Tab label="Album Lists" />
        <Tab label="Folder Lists" />
        <Tab label="Library" />
        <svelte:fragment slot="content">
            <TabContent>
                <DataTable
                    batchExpansion
                    sortable
                    zebra
                    size="compact"
                    stickyHeader
                    headers={[
                    //    { key: "dir", value: "Dir", width: "120px" },
                        { key: "name", value: "Name", width: "40%" },
                        { key: "tag_title", value: "Title", width: "40%" },
                    //    { key: "tag_album", value: "Album", width: "120px" },
                    //    { key: "tag_artist", value: "Artist", width: "120px" },
                    //    { key: "tag_track", value: "Track No.", width: "120px" },
                        { key: "like", value: "Like", width: "120px" },
                    ]}
                    rows={musics}
                    on:click:row={playMusic}
                />
            </TabContent>
            <TabContent>
                <DataTable
                    batchExpansion
                    sortable
                    zebra
                    size="compact"
                    stickyHeader
                    headers={[
                        { key: "tag_album", value: "Album" }
                    ]}
                    rows={albums}
                />
            </TabContent>
            <TabContent>
                <DataTable
                    batchExpansion
                    sortable
                    zebra
                    size="compact"
                    stickyHeader
                    headers={[
                        { key: "dir", value: "Dir" }
                    ]}
                    rows={folders}
                    on:click:row={playFolder}
                />
            </TabContent>
            <TabContent>
                <DataTable
                    zebra
                    size="compact"
                    stickyHeader
                    headers={[
                        { key: 'id', value: 'ID', width: '20%' },
                        { key: 'path', value: 'Path', width: '80%' },
                        { key: 'recursive', value: 'Rec', width: '20%' }
                    ]}
                    rows={libs}
                />
            </TabContent>
        </svelte:fragment>
        </Tabs>
      </Column>
    </Row>
    <Row>
        <Column>
            <Button size="small" icon={Play} iconDescription="Play Music" tooltipPosition="top" on:click={resumeMusic} />
            <Button size="small" icon={Stop} iconDescription="Stop Music" tooltipPosition="top" kind="secondary" on:click={stopMusic} />
            <Button size="small" icon={Pause} iconDescription="Pause" tooltipPosition="top" kind="secondary" on:click={pauseMusic} />
        </Column>
    </Row>
    <Row>
        <Column><Tile>{nowfile}</Tile></Column>
    </Row>
    <Row>
        <Column>
            <Slider
                min={min}
                max={max}
                maxLabel=""
                value={nowtime}
                fullWidth
                hideLabel
                hideTextInput 
            />
        </Column>
    </Row>
  </Grid>
</Content>


<style>
</style>