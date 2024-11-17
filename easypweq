#!/bin/bash

eqdir=~/pweq
pwdir=~/.config/pipewire/pipewire.conf.d/

nameprepend="EQ: "
fileprepend="pweq - "
devprepend="pweq_"

declare -A filtertype
filtertype[LSC]="bq_lowshelf"
filtertype[PK]="bq_peaking"
filtertype[HSC]="bq_highshelf"

function genheader(){
	name=$1
	fnum=0
	cat << EOF
context.modules =
[
  { name = libpipewire-module-filter-chain
    args = {
      node.description = "${nameprepend}${name}"
      media.name       = "${nameprepend}${name}"
      filter.graph = {
        nodes = [      
EOF
}

function gennode(){
# 1type 2preamp/ftype 3freq 4gain 5q
	case $1 in
		p)
			cat << EOF
          {
            type = builtin name = ${devprepend}preamp label = bq_highshelf
            control = { "Freq" = 0 "Q" = 1.0 "Gain" = $2 }
          }
EOF
			;;
		f)
			let fnum++
			cat << EOF
          {
            type = builtin name = ${devprepend}band_${fnum} label = ${filtertype[$2]}
            control = { "Freq" = $3 "Q" = $5 "Gain" = $4 }
          }
EOF
			;;
	esac
}

function genfooter(){
name=$1
	cat << EOF
        ]
       links = [
          { output = "${devprepend}preamp:Out" input = "${devprepend}band_1:In" }
EOF
	stop=$((fnum-1))
	for i in $(eval echo {1..$stop}); do
		fnum_next=$((i+1))
		cat << EOF
          { output = "${devprepend}band_${i}:Out" input = "${devprepend}band_${fnum_next}:In" }
EOF
	done
	cat << EOF
        ]
      }
      audio.channels = 2
      audio.position = [ FL FR ]
      capture.props  = {
        node.name    = "${devprepend}${name}_input"
        media.class  = Audio/Sink
      }
      playback.props = {
        node.name    = "${devprepend}${name}_output"
        node.passive = true
      }
    }
  }
]
EOF
}

function genfilter(){
	fname=$2
	name=$1
	genheader "$name"
	while read type p _ ftype _ freq _ _ gain _ _ q; do
		case $type in
			'Preamp:')
				gennode p $p
				;;
			'Filter')
				gennode f $ftype $freq $gain $q
				;;
		esac
	done < "$fname"
	genfooter "$name"
}

genfilter "HD599" "$eqdir/Sennheiser HD 599 ParametricEq.txt"
