# taglist配置

##  常用配置

let Tlist_Ctags_Cmd='/usr/bin/ctags'
let Tlist_Show_One_File = 1
let Tlist_Exit_OnlyWindow = 1
let Tlist_Use_SingleClick = 1
let Tlist_GainFocus_On_ToggleOpen = 1
let Tlist_WinWidth =35

" Show taglist window when open a file with vim
"let Tlist_Auto_Open = 1

" Close taglist window when select a tag
"let Tlist_Close_On_Select = 1

" when show tags of all open files, fold the other backgroud files
"let Tlist_File_Fold_Auto_Close = 1

map <leader>tt: TlistToggle<cr>

