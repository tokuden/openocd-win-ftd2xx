# Welcome to OpenOCD-win-ftd2xx

OpenOCD-win-ftd2xx��OpenOCD���AWindows�œ��삷��悤�Ƀr���h�������̂ł��B
�I�[�v���\�[�X��libusb��libftdi�ł͂Ȃ��AFTDI�Ђ̃v���v���C�G�^����FTD2XX�h���C�o���g���ē��삷��悤�ɏ��������܂����B

## ����
* �I�[�v���\�[�X��libftdi��libusb�ł͂Ȃ��AFTDI������FTD2XX�œ����悤�ɃJ�X�^�}�C�Y���Ă��邽�߁AZadig��libusb�ɓ���ւ���K�v���Ȃ�
* FTDI�Ђ�FT232H��FT2232D/H�AFT4232H�Ȃǂ�MPSSE���g����USB-JTAG�Ŕėp�I�Ɏg����B
* ���̑���USB-JTAG�P�[�u��(CMSIS-DAP��J-LINK)�ɂ͑Ή����Ă��Ȃ��B
* Windows��EXE�t�@�C���Œ񋟂���邽�߁A�ڂ����Ȃ��l�ɁuWSL��p�ӂ��Ă��������v�ƌ����Ă����点��S�z���Ȃ�
* �ŐV��OpenOCD 0.12���x�[�X�ɂ��Ă���

�^�[�Q�b�g�́AZYNQ7000��UltraScale+�ARaspberry Pi 4�ɐڑ��ł��邱�Ƃ��m�F���Ă��܂��B
Windows�̃l�C�e�B�u�A�v���Ƃ��ē��삵�܂��̂ŁAWSL�Ȃǂ̉��z���A�R���e�i�͕s�v�ł��B

## �_�E�����[�h�ƃC���X�g�[��
https://github.com/tokuden/openocd-win-ftd2xx/OpenOCD-Win-FTD2XX.zip ���_�E�����[�h���ĉ𓀂��Ă��������B

## �g����
MSDOS�v�����v�g���J���āA
`openocd.exe -f ft2232h.cfg -f zynq7000.cfg`
�̂悤�ɓ��͂��܂��B

�\�������܂��Ă�����o�b�`�t�@�C����������ق����������Ǝv���܂��B
���d��MPSSE-JTAG�P�[�u�����g�p����ꍇ�́A-f np1167.cfg�Ǝw�肵�Ă��������B
�N������ƈȉ��̂悤�ȉ�ʂɂȂ�܂��B

CPU�̃f�o�b�O������ɂ�TELNET��localhost:4444�Ƀ��O�C��������AGDB�Ń����[�g�ڑ�����K�v������܂��B
�Ⴆ�΁A����������0�Ԓn�̓��e���_���v����ɂ́ATeraTerm���g����localhost:4444��TELNET�ڑ����A
'''
targets
halt
mdw phys 0 0x100
redume
'''

