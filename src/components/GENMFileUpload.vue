<template>
  <label>
    <input type="file" id="input" accept=".genm" @change="fileAdded" />
  </label>
</template>

<script>
import genmdmMapping from "../genmdm-mapping";
import mapToCCRange from "../utils/map-to-cc-range";
/* GENM format
 * ----------
 * A .GENM file is used to store GenMDM patch files.
 *
 * Each patch file can store up to 128 different instrument settings for an FM channel of a GenMDM interface.
 *
 * A .GENM file is in a standard text format.
 *
 * Each instrument is stored in the following way:
 * [Instrument Index], [Algorithm] [LFO FM] [LFO AM] [FM Feedback] [Panning] [OP1 Level] [OP2 Level] [OP3 Level] [OP4 Level] [OP1 Detune] [OP2 Detune] [OP3 Detune] [OP4 Detune] [OP1 Attack] [OP2 Attack] [OP3 Attack] [OP4 Attack] [OP1 Decay 1] [OP2 Decay 1] [OP3 Decay 1] [OP4 Decay 1] [OP1 Decay 2] [OP2 Decay 2] [OP3 Decay 2] [OP4 Decay 2] [OP1 Mul] [OP2 Mul] [OP3 Mul] [OP4 Mul] [OP1 Rate Scaling] [OP2 Rate Scaling] [OP3 Rate Scaling] [OP4 Rate Scaling] [OP1 Amp2] [OP2 Amp2] [OP3 Amp2] [OP4 Amp2] [OP1 Release] [OP2 Release] [OP3 Release] [OP4 Release] [OP1 LFO Enable] [OP2 LFO Enable] [OP3 LFO Enable] [OP4 LFO Enable] [OP1 SSG Data] [OP2 SSG Data] [OP3 SSG Data] [OP4 SSG Data] [Instrument Name Symbol String];
 *
 * • Instrument index is indexed at 0
 * • Instrument index does not need need to be in order in the file
 * • TL is given as 127 - TL for each level parameter for each operator
 * • LFO enable is 0 for not enabled, 1 for enabled
 * • All other data fields are given as actual register data, not unscaled raw data
 * • Instrument name can be up to 256 characters long
 */

const labels = [
  "algorithm",
  "lfoFm",
  "lfoAm",
  "fmFeedback",
  "panning",
  "op1Level",
  "op2Level",
  "op3Level",
  "op4Level",
  "op1Detune",
  "op2Detune",
  "op3Detune",
  "op4Detune",
  "op1Attack",
  "op2Attack",
  "op3Attack",
  "op4Attack",
  "op1Decay1",
  "op2Decay1",
  "op3Decay1",
  "op4Decay1",
  "op1Decay2",
  "op2Decay2",
  "op3Decay2",
  "op4Decay2",
  "op1Mul",
  "op2Mul",
  "op3Mul",
  "op4Mul",
  "op1RateScaling",
  "op2RateScaling",
  "op3RateScaling",
  "op4RateScaling",
  "op1Amp2",
  "op2Amp2",
  "op3Amp2",
  "op4Amp2",
  "op1Release",
  "op2Release",
  "op3Release",
  "op4Release",
  "op1LfoEnable",
  "op2LfoEnable",
  "op3LfoEnable",
  "op4LfoEnable",
  "op1SsgData",
  "op2SsgData",
  "op3SsgData",
  "op4SsgData",
  "instrumentNameSymbolString"
];

export default {
  methods: {
    fileAdded(e) {
      const {
        files: { 0: file }
      } = e.target;

      const reader = new FileReader();
      reader.onload = () => {
        const text = reader.result;

        this.parseGenmData(text);
      };
      reader.readAsText(file);
    },

    parseGenmData(text) {
      let split = text.split(";");
      split = split.filter(item => item.trim().length);
      split = split.slice(0, 128);

      const ordered = [];

      for (let i = 0; i < split.length; ++i) {
        const item = split[i];
        const splitData = item.split(", ");
        const index = parseInt(splitData.shift(), 10);

        ordered[index] = splitData[0];
      }

      const data = [];

      for (let i = 0; i < ordered.length; ++i) {
        const item = ordered[i].split(" ");
        const parsed = {};

        for (let j = 0; j < item.length; ++j) {
          if (j < item.length - 1) {
            parsed[labels[j]] = parseInt(item[j], 10);
          } else {
            parsed[labels[j]] = item[j];
          }
        }

        data[i] = parsed;
      }

      const instrumentData = [];

      for (let i = 0; i < data.length; ++i) {
        const instrument = data[i];

        const ccData = {
          // algorithm
          14: mapToCCRange(
            instrument["algorithm"],
            genmdmMapping[14].range - 1
          ),

          // feedback
          15: mapToCCRange(
            instrument["fmFeedback"],
            genmdmMapping[15].range - 1
          ),

          // lfo fm
          75: mapToCCRange(instrument["lfoFm"], genmdmMapping[75].range - 1),

          // lfo am
          76: mapToCCRange(instrument["lfoAm"], genmdmMapping[76].range - 1),

          // panning
          77: mapToCCRange(instrument["panning"], genmdmMapping[77].range - 1)
        };

        for (let i = 0; i < 4; ++i) {
          const j = i + 1;

          // Multiple
          ccData[20 + i] = mapToCCRange(
            instrument[`op${j}Mul`],
            genmdmMapping[20].range - 1
          );
          // instrument[`op${j}Mul`]

          // Detune
          ccData[24 + i] = mapToCCRange(
            instrument[`op${j}Detune`],
            genmdmMapping[24].range - 1
          );

          // Total Level
          ccData[16 + i] = mapToCCRange(
            instrument[`op${j}Level`],
            genmdmMapping[16].range - 1
          );

          // Rate Scaling
          ccData[39 + i] = mapToCCRange(
            instrument[`op${j}RateScaling`],
            genmdmMapping[39].range - 1
          );

          // Attack Rate
          ccData[43 + i] = mapToCCRange(
            instrument[`op${j}Attack`],
            genmdmMapping[43].range - 1
          );

          // First Decay Rate
          ccData[47 + i] = mapToCCRange(
            instrument[`op${j}Decay1`],
            genmdmMapping[47].range - 1
          );

          // Secondary Decay Rate
          ccData[51 + i] = mapToCCRange(
            instrument[`op${j}Decay2`],
            genmdmMapping[51].range - 1
          );

          // Release Rate
          ccData[59 + i] = mapToCCRange(
            instrument[`op${j}Release`],
            genmdmMapping[59].range - 1
          );

          // Secondary Amplitude Level
          ccData[55 + i] = mapToCCRange(
            instrument[`op${j}Amp2`],
            genmdmMapping[55].range - 1
          );

          // SSG-EG Operator
          ccData[90 + i] = mapToCCRange(
            instrument[`op${j}SsgData`],
            genmdmMapping[90].range - 1
          );

          // OP LFO Enable
          ccData[70 + i] = mapToCCRange(
            instrument[`op${j}LfoEnable`],
            genmdmMapping[70].range - 1
          );
        }

        instrumentData.push({
          name: instrument.instrumentNameSymbolString,
          data: ccData
        });
      }

      this.$store.dispatch("setPatches", instrumentData);
    }
  }
};
</script>

<style scoped>
input {
  display: none;
}

label::before {
  appearance: button;
  padding: 1px 6px;

  content: "Load GENM";
  display: inline-block;
  border: 1px solid var(--foreground-color);
  color: var(--foreground-color);

  outline: none;
  white-space: nowrap;
  -webkit-user-select: none;
  font-size: 10pt;
  text-transform: uppercase;

  display: inline-block;
  text-align: center;
  width: -webkit-fill-available;
}
</style>
