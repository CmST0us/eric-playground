这里pts的base_clock都是按照1000（毫秒）计算，如果复用到ts里，base_clock是90k，所以还应该再乘以90。关于H264中sps里面记录的帧率是实际帧率的2倍，包括slice里面的pic_order_cnt_lsb也是2倍递增，我推测可能是编码按照分场（顶场、底场）编码所致。

H264的ES原始数据一般是以NAL（Network Abstract Layer）的格式存在。可以直接用于文件存储和网络传输。每一个NALU(Network Abstract Layer Unit)数据，是由数据头+RBSP数据组成。
首先需要将数据流，分割成一个一个独立的NALU数据。
接着获取NALU的nal_type，i_nal_type的值等于0x7表示这个nalu是个sps数据包。找到并解析这个sps数据包，里面包含有非常重要的帧率信息
time_scale/num_units_in_tick=fps
然后根据nal_type判断slice（H264中的slice类似一个视频帧FRAME的概念）。其中nal_type值小于0x1，或大于0x5，表示这个NALU属于一个slice。

<code>
// 检查是否是slice  
if ( i_nal_type < 1/*NAL_SLICE*/ || i_nal_type > 5/*NAL_SLICE_IDR*/ )  
   // 找到slice!!!!! 
</code>

在找到slice的NALU后，可以逐字节将NALU的数据与0x80进行与运算，结果为真表示这个slice（视频帧FRAME）的结束位置。

<code>
// 判断是否帧结束
for (uint32_t i = 3; i < nal_length; i++)  {      
    if (p_nal[i] & 0x80)      {    
    // 找到frame_begin!!!!上一帧frame的结束,下一帧frame的开始   
   } 
}  
</code>

上面的这个代码是摘抄自FFMPEG。他实际作用是判断slice里面的first_mb_in_slice，即第1个宏块在slice中的位置， 如果是一帧开始，这个字段的值肯定是标识第1个宏块。因此，也可以完整解析slice的头部信息，解析出first_mb_in_slice，如果是 0（注意：这是1个哥伦布数值），即这个NALU是一帧的开始。
为什么这里的代码是逐字节判断0x80？我额外写点某大神的名言：程序猿不是十万个为什么，不是维基猿，程序猿是需求猿。如果某程序猿已经着手开始研究如何解析slice头部格式，他很自然的不会有这个疑问。
另外通过nal_type以及silice_type也可以判断出帧结束位置，VLC里面的代码就是这么干。
解析到位于帧结束位置的NALU，就可以判断出每一帧（slice）的开始和结尾。解析slice的slice_type，根据slice_type，可以判断出这个slice的IPB类型。

<code>
// 根据slice类型判断帧类型  switch(slice.i_slice_type)  {  case 2: case 7:   case 4: case 9:      *p_flags = 0x0002/*BLOCK_FLAG_TYPE_I*/;      break;  case 0: case 5:  case 3: case 8:      *p_flags = 0x0004/*BLOCK_FLAG_TYPE_P*/;     break;  case 1:  case 6:      *p_flags = 0x0008/*BLOCK_FLAG_TYPE_B*/;      break;  default:      *p_flags = 0;      break;  } 
</code>

从现在开始，就有两种办法来计算PTS了。
方法一、根据前后帧的IPB类型，可以得知帧的实际显示顺序，使用前面获取的sps信息中的帧率，以及帧计数frame_count即可计算出PTS。此方法需要做几帧缓存（一般缓存一个group的长度）。
I  P  B  B  I  P  B  
B  I  P  B  ... 帧类型
1  2  3  4  5  6  7  8  9  10 11 ... 第几帧
1  4  2  3  5  8  6  7  9  12 10 ... 帧显示顺序
一个I帧与下一个I帧之间，是一个group。
从上图可见，P类型的帧的显示顺序，是排在后面最后一个B帧之后。
所以要获取第7帧的pts，起码要知道他下一帧的类型，才能得知他的显示顺序。
第8帧的pts=1000（毫秒）*7（帧显示顺序）*帧率
方法二、每一个slice的信息里面，都记录有pic_order_cnt_lsb，当前帧在这个group中的显示顺序。通过这个pic_order_cnt_lsb，可以直接计算出当前帧的PTS。此方法不需要做帧缓存。
计算公式：
pts=1000*(i_frame_counter + pic_order_cnt_lsb)*(time_scale/num_units_in_tick)
i_frame_counter是最近一次I帧位置的帧序，通过I帧计数+当前group中的帧序，得到帧实际显示序列位置，乘上帧率，再乘上1000（毫秒）的base_clock（基本时钟频率），得到PTS。
I  P  B  B  I  P  B  B  
I  P  B  ... 帧类型
1  2  3  4  5  6  7  8  9  10 11 ... 第几帧
1  4  2  3  5  8  6  7  9  12 10 ... 帧显示顺序
0  6  2  4  0  6  2  4  0  6  
2  ... pic_order_cnt_lsb
细心一点可以注意到，在上图，slice里面的pic_order_cnt_lsb是以2进行递增。
通常H264里面的sps中记录的帧率，也是实际帧率的2倍time_scale/num_units_in_tick=fps*2
因此，实际的计算公式应该是这样
pts=1000*(i_frame_counter*2+pic_order_cnt_lsb)* (time_scale/num_units_in_tick)
或者是
pts=1000*(i_frame_counter+pic_order_cnt_lsb/2)* (time_scale/num_units_in_tick/2)
所以，第11帧的pts应该是这么计算
1000*(9*2+2)*(time_scale/num_units_in_tick)
结束语：
这里pts的base_clock都是按照1000（毫秒）计算，如果复用到ts里，base_clock是90k，所以还应该再乘以90。
题外话：关于H264中sps里面记录的帧率是实际帧率的2倍，包括slice里面的pic_order_cnt_lsb也是2倍递增，我推测可能是编码按照分场（顶场、底场）编码所致。另外我注意到sps信息中的offset_for_top_to_bottom_field字段，从命名上，貌似是可以用 来标记是否逐场，还是分奇偶场编码。以上都属猜测，有请高人解惑。

---------------------

本文来自 daydayup 的CSDN 博客 ，全文地址请点击：https://blog.csdn.net/a511244213/article/details/49098973?utm_source=copy 