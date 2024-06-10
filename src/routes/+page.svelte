<script>
    import "carbon-components-svelte/css/g90.css";

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
        Pagination,
    } from "carbon-components-svelte";
    import Play from "carbon-icons-svelte/lib/Play.svelte";
    import Stop from "carbon-icons-svelte/lib/Stop.svelte";
    import Pause from "carbon-icons-svelte/lib/Pause.svelte";
    import Mute from "carbon-icons-svelte/lib/VolumeMute.svelte";

    let value = "";
    let musics = [];
    let page = 1;
    let pageSize = 20;
    let albums = [];
    let folders = [];
    let libs = [];
    let queue = [];
    let min = 0;
    let max = 0;
    let nowtime = 0;
    let nowplaying = {
        title: '',
        album: '',
        artist: '',
        track: ''
    };

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

    async function deleteQueue() {
        let res = await fetch('/deletequeue.json', {
            method: 'POST'
        });
        await getQueue();
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

    async function muteMusic() {
        fetch('/mute.json');
    }

    function hiddenSliderCnt() {
        let ts = document.querySelectorAll('span.bx--slider__range-label');
        if(ts) {
            ts.forEach(function(t) {
                t.style.display = 'none';
            });
            let t = document.querySelectorAll('span.bx--slider__range-label')[1];
            t.insertAdjacentHTML('afterend', '<span class="bx--slider__range-label playtime">0</span>');
        }
    }

    function updatePlaytime(val) {
        document.querySelector('span.playtime').innerText = Math.floor(val.duration/60) + ':' + Math.floor(val.duration%60);
    }

    setTimeout(async function() {
        $: console.log('Init');
        hiddenSliderCnt();
        getLib();
        getMusic();
        getQueue();
        streamStatus();
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
                })();
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
        <Tab label="Music" />
        <Tab label="Album" />
        <Tab label="Folder" />
        <Tab label="Queue" />
        <Tab label="Library" />
        <svelte:fragment slot="content">
            <TabContent>
                <DataTable
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
                        { key: 'path', value: 'Path', width: '80%' }
                    ]}
                    rows={queue}
                />
            </TabContent>
            <TabContent>
                <DataTable
                    zebra
                    size="compact"
                    stickyHeader
                    headers={[
                        { key: 'id', value: 'ID', width: '20%' },
                        { key: 'path', value: 'Path', width: '60%' },
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
            <Button size="small" icon={Mute} iconDescription="Mute" tooltipPosition="top" kind="secondary" on:click={muteMusic} />
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