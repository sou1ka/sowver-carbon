<script>
    import "carbon-components-svelte/css/g90.css";

    import {
        Header,
        HeaderUtilities,
        HeaderAction,
        SkipToContent,
        HeaderSearch,
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
        Pagination,
        TextInput,
        Checkbox
    } from "carbon-components-svelte";
    import Play from "carbon-icons-svelte/lib/PlayFilled.svelte";
    import Stop from "carbon-icons-svelte/lib/Stop.svelte";
    import Pause from "carbon-icons-svelte/lib/Pause.svelte";
    import Mute from "carbon-icons-svelte/lib/VolumeMute.svelte";
    import Next from "carbon-icons-svelte/lib/ArrowRight.svelte";
    import Prev from "carbon-icons-svelte/lib/ArrowLeft.svelte";
    import Loop from "carbon-icons-svelte/lib/Loop.svelte";
    import Erase from "carbon-icons-svelte/lib/Erase.svelte";

    let isOpen = false;
    let value = "";
    let musics = [];
    let page = 1;
    let pageSize = 20;
    let albums = [];
    let folders = [];
    let libs = [];
    let queue = [];
    let playlists = [];
    let libstatus = '';
    let min = 0;
    let max = 0;
    let isVol = false;
    let nowtime = 0;
    let nowplaying = {
        title: '',
        album: '',
        artist: '',
        track: ''
    };
    let addlibpath = '';
    let addlibrec = true;
    let saveplaylistpath = '';

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

    async function insertLib() {
        let params = new URLSearchParams();
        params.append('path', addlibpath);
        params.append('recursive', (addlibrec ? "1": "0"));

        await fetch('/insertlib.json', {
            method: 'POST',
            body: params
        });

        addlibpath = '';
        addlibrec = true;

        await getLib();
    }

    async function getQueue() {
        let res = await fetch('/queue.json', {
            method: 'POST'
        });
        queue = await res.json();
    }

    async function insertQueue(row) {
        let params = new URLSearchParams();
        params.append('path', row.detail.path);
        params.append('name', row.detail.name);

        fetch('/insertqueue.json', {
            method: 'POST',
            body: params
        });

        if(queue.length == 0) {
            playMusic(row);
        }

        await getQueue();
    }

    async function deleteQueue(row) {
        let params = new URLSearchParams();
        params.append('id', row.detail.id);

        await fetch('/deletequeue.json', {
            method: 'POST',
            body: params
        });
        await getQueue();
    }

    async function getPlaylist() {
        fetch('/playlists.json', {
            method: 'POST'
        }).then(async function(res) {
            let json = await res.json();
            let cnt = 0;
            for(let l of json) {
                l.id = cnt;
                cnt++;
            }
            playlists = json;
        });
    }

    async function playPlaylist(row) {
        let params = new URLSearchParams();
        params.append('path', row.detail.path);
        params.append('name', row.detail.name);

        fetch('/loadplaylist.json', {
            method: 'POST',
            body: params
        });
    }

    async function savePlaylist() {
        let params = new URLSearchParams();
        params.append('path', saveplaylistpath);

        await fetch('/saveplaylist.json', {
            method: 'POST',
            body: params
        });

        saveplaylistpath = '';
    }

    async function resumeMusic() {
        fetch('/resume.json');
    }

    async function stopMusic() {
        await fetch('/stop.json');
        getQueue();
    }

    async function pauseMusic() {
        fetch('/togglepause.json');
    }

    async function muteMusic() {
        fetch('/mute.json');
    }

    async function nextMusic() {
        fetch('/next.json');
    }

    async function prevMusic() {
        fetch('/prev.json');
    }

    async function setVolume(sld) {
        if(isVol) {
            isVol = false;
            let params = new URLSearchParams();
            params.append('volume', Number(sld.detail));

            fetch('/volume.json', {
                method: 'POST',
                body: params
            });

        } else {
            isVol = true;
        }
    }

    async function loopMusic() {
        fetch('/loop.json');
    }

    async function clearloopMusic() {
        fetch('/clearloop.json');
    }

    function hiddenSliderCnt() {
        let ts = document.querySelectorAll('span.bx--slider__range-label');
        if(ts) {
            ts[2].style.display = 'none';
            ts[3].style.display = 'none';
            ts[3].insertAdjacentHTML('afterend', '<span class="bx--slider__range-label playtime">0</span>');
        }
    }

    function updatePlaytime(val) {
        document.querySelector('span.playtime').innerText = Math.floor(val.duration/60) + ':' + Math.floor(val.duration%60);
    }

    async function readlib() {
        fetch('/readlib.json');
        readLibstatus();
    }

    setTimeout(async function() {
        $: console.log('Init');
        hiddenSliderCnt();
        getLib();
        getMusic();
        getQueue();
        getPlaylist();
        streamStatus();
        readLibstatus();
    }, 1000);

    function streamStatus() {
        let stream = new EventSource('/stream');
        stream.addEventListener('message', function(e) {
            if(!e.data) { return; }
            let st = JSON.parse(e.data);

            if(st.duration) {
                max = Math.ceil(st.duration);
                nowtime = Math.ceil(st.timePos);
                nowplaying = st.play;
                nowplaying.album = nowplaying.album || '';
                nowplaying.track = nowplaying.track || '';
                nowplaying.title = nowplaying.title || '';
                nowplaying.artist = nowplaying.artist || '';
                updatePlaytime(st);

            } else {
                st = 0;
                max = 0;
                nowtime = 0;
                nowplaying = {
                    title: '',
                    album: '',
                    artist: '',
                    track: ''
                };
/*
                (async function() {
                    if(queue) {
                        await deleteQueue();
                        if(queue && queue.length > 0 && queue[0]) {
                            await playMusic({
                                detail: {
                                    path: queue[0].path,
                                    name: ''
                                }
                            });
                        }
                    }
                })();*/
            }
        });
    }

    function readLibstatus() {
        let stream = new EventSource('/readlibstatus');
        stream.addEventListener('message', function(e) {
            if(e.data == 'Finalize') {
                stream.close();
                return;
            }

            libstatus = e.data;
        });
    }
</script>

<Header company="MONOBOX" platformName="Sowver">
    <svelte:fragment slot="skip-to-content">
        <SkipToContent />
    </svelte:fragment>
    <HeaderUtilities>
        <HeaderAction bind:isOpen preventCloseOnClickOutside=true>
            <DataTable
                zebra
                size="compact"
                headers={[
                    { key: 'id', value: 'ID', width: '20%' },
                    { key: 'filename', value: 'Path', width: '80%' }
                ]}
                rows={queue}
            />
        </HeaderAction>
    </HeaderUtilities>
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
        <Tab label="Music" />
        <Tab label="Album" />
        <Tab label="Folder" />
        <Tab label="Queue" />
        <Tab label="Playlist" />
        <Tab label="Library" />
        <svelte:fragment slot="content">
            <TabContent>
                <DataTable
                    sortable
                    zebra
                    size="compact"
                    headers={[
                    //    { key: "dir", value: "Dir", width: "120px" },
                        { key: "name", value: "Name", width: "30%" },
                        { key: "tag_title", value: "Title", width: "30%" },
                        { key: "tag_album", value: "Album", width: "120px" },
                        { key: "tag_artist", value: "Artist", width: "120px" },
                    //    { key: "tag_track", value: "Track No.", width: "120px" },
                        { key: "like", value: "Like", width: "120px" },
                    ]}
                    {pageSize}
                    {page}
                    rows={musics}
                    on:click:row={insertQueue}
                />
                <Pagination
                    bind:pageSize
                    bind:page
                    totalItems={musics.length}
                    pageSizeInputDisabled
                />
            </TabContent>
            <TabContent>
                <DataTable
                    batchExpansion
                    sortable
                    zebra
                    size="compact"
                    headers={[
                        { key: "tag_album", value: "Album" },
                        { key: "tag_artist", value: "Artist", width: "120px" },
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
                    headers={[
                        { key: 'id', value: 'ID', width: '20%' },
                        { key: 'filename', value: 'Path', width: '80%' }
                    ]}
                    rows={queue}
                    on:click:row={deleteQueue}
                />
                <Grid>
                    <Row>
                        <Column sm={{span: 2}}>
                            <TextInput hideLabel placeholder="Input save path" bind:value={saveplaylistpath}></TextInput>
                        </Column>
                        <Column>
                            <Button on:click={savePlaylist}>Save Playlist.</Button>
                        </Column>
                    </Row>
                </Grid>
            </TabContent>
            <TabContent>
                <DataTable
                    zebra
                    size="compact"
                    headers={[
                        { key: 'name', value: 'Name', width: '20%' },
                        { key: 'path', value: 'Path', width: '60%' }
                    ]}
                    rows={playlists}
                    on:click:row={playPlaylist}
                />
            </TabContent>
            <TabContent>
                <DataTable
                    zebra
                    size="compact"
                    headers={[
                        { key: 'id', value: 'ID', width: '20%' },
                        { key: 'path', value: 'Path', width: '60%' },
                        { key: 'recursive', value: 'Rec', width: '20%' }
                    ]}
                    rows={libs}
                />
                <Grid>
                    <Row>
                        <Column sm={{span: 2}}>
                            <TextInput hideLabel placeholder="Input music path" bind:value={addlibpath}></TextInput>
                        </Column>
                        <Column>
                            <Checkbox labelText="Recursive" bind:checked={addlibrec}></Checkbox>
                        </Column>
                        <Column>
                            <Button on:click={insertLib}>Add Lib.</Button>
                        </Column>
                    </Row>
                </Grid>
                <Button kind="ghost" on:click={readlib}>Reload Libraly.</Button>
                <Tile>{libstatus}</Tile>
            </TabContent>
        </svelte:fragment>
        </Tabs>
      </Column>
    </Row>
    <Row>
        <Column sm={{span: 1}}>
            <Button size="small" icon={Play} iconDescription="Play Music" tooltipPosition="top" on:click={resumeMusic} />
            <Button size="small" icon={Prev} iconDescription="Prev Music" tooltipPosition="top" kind="secondary" on:click={prevMusic} />            
            <Button size="small" icon={Next} iconDescription="Next Music" tooltipPosition="top" kind="secondary" on:click={nextMusic} />            
            <Button size="small" icon={Pause} iconDescription="Pause" tooltipPosition="top" kind="secondary" on:click={pauseMusic} />
            <Button size="small" icon={Loop} iconDescription="Loop" tooltipPosition="top" kind="secondary" on:click={loopMusic} />
            <Button size="small" icon={Erase} iconDescription="Clear Loop" tooltipPosition="top" kind="secondary" on:click={clearloopMusic} />
            <Button size="small" icon={Mute} iconDescription="Mute" tooltipPosition="top" kind="secondary" on:click={muteMusic} />
            <Button size="small" icon={Stop} iconDescription="Stop Music" tooltipPosition="top" kind="secondary" on:click={stopMusic} />
        </Column>
        <Column sm={{span: 3}}>
            <Slider
                min={0}
                max={100}
                labelText="Volume"
                value={50}
                hideTextInput
                on:change={setVolume}
            />
        </Column>
    </Row>
    <Row>
        <Column><Tile>{nowplaying.album} {nowplaying.track} - {nowplaying.title} / {nowplaying.artist}</Tile></Column>
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